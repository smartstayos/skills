# Client di riferimento

Un client minimo ma completo: autenticazione da variabile d'ambiente, paginazione, retry con attesa
crescente solo dove è sicuro, errori con `request_id`, `202 pending_approval` distinto dal
successo. Adattalo al progetto dell'utente invece di spargere `fetch` nel codice.

Il retry automatico vale per le letture (`GET`) e per le scritture che portano una chiave di
idempotenza (`idempotency_key` o `client_session_id`): un `502`/`504` del proxy può arrivare
DOPO che il server ha eseguito l'operazione, e senza chiave il secondo tentativo la duplica.

## TypeScript (Node 18+, Deno, Bun)

```ts
const BASE = process.env.SMARTSTAY_API_BASE ?? 'https://api.smartstayos.com';
const KEY = process.env.SMARTSTAY_API_KEY;
if (!KEY) throw new Error('SMARTSTAY_API_KEY mancante');

export class SmartStayApiError extends Error {
  constructor(public status: number, public code: string, message: string, public requestId?: string) {
    super(`${status} ${code}: ${message}${requestId ? ` (request_id ${requestId})` : ''}`);
  }
}

/** Risultato di un'azione sensibile: ricevuta, non eseguita. */
export interface PendingApproval { status: 'pending_approval'; approval_id: string }

const RETRY = new Set([429, 502, 503, 504]);

/** Ritentare è sicuro solo per una lettura o per una scrittura con chiave di idempotenza. */
function safeToRetry(method: string, body: unknown): boolean {
  if (method === 'GET') return true;
  const b = (body ?? {}) as Record<string, unknown>;
  return typeof b.idempotency_key === 'string' || typeof b.client_session_id === 'string';
}

export async function call<T>(method: string, path: string, opts: { query?: Record<string, string | number | boolean | undefined>; body?: unknown } = {}): Promise<{ status: number; data: T; pagination?: { limit: number; offset: number; returned: number; has_more: boolean } }> {
  const url = new URL(BASE + path);
  for (const [k, v] of Object.entries(opts.query ?? {})) if (v !== undefined) url.searchParams.set(k, String(v));
  for (let attempt = 0; ; attempt++) {
    const res = await fetch(url, {
      method,
      headers: { Authorization: `Bearer ${KEY}`, ...(opts.body ? { 'Content-Type': 'application/json' } : {}) },
      body: opts.body ? JSON.stringify(opts.body) : undefined,
    });
    const json = await res.json().catch(() => ({}));
    if (res.ok) return { status: res.status, data: json.data as T, pagination: json.pagination };
    if (RETRY.has(res.status) && attempt < 4 && safeToRetry(method, opts.body)) { await new Promise((r) => setTimeout(r, 2 ** attempt * 1000)); continue; }
    throw new SmartStayApiError(res.status, json.error?.code ?? 'unknown', json.error?.message ?? res.statusText, json.request_id ?? res.headers.get('X-Request-Id') ?? undefined);
  }
}

/** Tutte le pagine di una lista, 100 alla volta. */
export async function* paginate<T>(path: string, query: Record<string, string | number | boolean | undefined> = {}): AsyncGenerator<T> {
  for (let offset = 0; ; ) {
    const page = await call<T[]>('GET', path, { query: { ...query, limit: 100, offset } });
    yield* page.data;
    if (!page.pagination?.has_more) return;
    offset += page.pagination.returned;
  }
}

/** Azione con esito distinto: eseguita (200) o in attesa di approvazione (202). */
export async function action<T>(name: string, body: Record<string, unknown>): Promise<{ done: true; data: T } | { done: false; pending: PendingApproval }> {
  const res = await call<T | PendingApproval>('POST', `/v1/actions/${name}`, { body });
  return res.status === 202 ? { done: false, pending: res.data as PendingApproval } : { done: true, data: res.data as T };
}
```

Uso:

```ts
for await (const booking of paginate('/v1/bookings', { from: '2026-12-01', to: '2026-12-07' })) console.log(booking);

const key = crypto.randomUUID(); // UNA per operazione: salvala con l'operazione e riusala nei retry
const res = await action('create_direct_booking', { kind: 'block', property_id, room_type_id, check_in, check_out, idempotency_key: key });
```

## Python (3.9+, requests)

```python
import os, time, uuid, requests

BASE = os.environ.get("SMARTSTAY_API_BASE", "https://api.smartstayos.com")
KEY = os.environ["SMARTSTAY_API_KEY"]
RETRY = {429, 502, 503, 504}

class SmartStayApiError(Exception):
    def __init__(self, status, code, message, request_id=None):
        super().__init__(f"{status} {code}: {message} (request_id {request_id})")
        self.status, self.code, self.request_id = status, code, request_id

def safe_to_retry(method, body):
    # Una lettura, o una scrittura con chiave di idempotenza: altrimenti un 502/504 arrivato dopo
    # l'esecuzione duplicherebbe l'operazione.
    return method == "GET" or bool(body and (body.get("idempotency_key") or body.get("client_session_id")))

def call(method, path, query=None, body=None):
    for attempt in range(5):
        res = requests.request(method, BASE + path, params=query, json=body, timeout=30,
                               headers={"Authorization": f"Bearer {KEY}"})
        try:
            payload = res.json() if res.content else {}
        except ValueError:  # un 502 del proxy può rispondere HTML
            payload = {}
        if res.ok:
            return res.status_code, payload
        if res.status_code in RETRY and attempt < 4 and safe_to_retry(method, body):
            time.sleep(2 ** attempt)
            continue
        err = payload.get("error", {})
        raise SmartStayApiError(res.status_code, err.get("code", "unknown"), err.get("message", res.reason),
                                payload.get("request_id") or res.headers.get("X-Request-Id"))

def paginate(path, **query):
    offset = 0
    while True:
        _, page = call("GET", path, {**query, "limit": 100, "offset": offset})
        yield from page["data"]
        if not page.get("pagination", {}).get("has_more"):
            return
        offset += page["pagination"]["returned"]

def action(name, body):
    status, payload = call("POST", f"/v1/actions/{name}", body=body)
    return {"done": status != 202, "data": payload["data"]}
```

## Cosa NON fare nel client

- Ritentare un `400`, `403`, `404` o `409`: darà lo stesso esito.
- Ritentare in automatico una scrittura senza chiave (task, contatto, fattura manuale, invito al
  check-in): dopo un timeout, prima si controlla con una lettura se è già avvenuta.
- Generare una `idempotency_key` nuova a ogni tentativo: annulla la protezione dai doppioni.
- Chiedere più di 100 righe per pagina o leggere tutto a ogni giro: per una sincronizzazione
  periodica filtra per data (`from`/`to`) e salva dove sei arrivato.
- Trattare `202` come fatto: è in attesa di una persona.
