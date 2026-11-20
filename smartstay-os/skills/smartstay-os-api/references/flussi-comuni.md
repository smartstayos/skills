# Flussi comuni

Ricette verificate sugli endpoint reali. Prima di usarne una, controlla nella spec i campi
dell'operazione: le ricette dicono il percorso, la spec dice tutto il resto.

## Arrivi della settimana con lo stato del check-in

Scope: `bookings:read`, `checkin:read` (e `operations:read` per lo stato completo).

1. `GET /v1/bookings?from=AAAA-MM-GG&to=AAAA-MM-GG` (paginata). Le cancellate sono escluse; per
   averle, `include_cancelled=true`. Filtro per casa: `property_id`.
2. Per ogni prenotazione: `GET /v1/bookings/{id}/checkin` (fase del check-in, mai il link del
   portale ospite), oppure `GET /v1/bookings/{id}?operational_state=true` per check-in, cauzione,
   tassa e pulizie in una risposta.

## Pulizie del giorno con l'addetto

Scope: `housekeeping:read` (+ `staff:read` per i nomi).

`GET /v1/cleanings?from=AAAA-MM-GG&to=AAAA-MM-GG`: ogni pulizia ha `assigned_staff_id`, tipo,
`not_before_time` / `ready_by_time`, ospiti e letti in arrivo, prenotazioni in uscita e in entrata.
Una pulizia con `assigned_staff_id` vuoto prima di un arrivo è la prima cosa da segnalare. Il nome
dell'addetto: `POST /v1/actions/get_staff` con `staff_id`.

## Tassa di soggiorno per mese

Scope: `tourist_tax:read`.

`GET /v1/resources/tourist_tax_ledger` (paginata, filtro `property_id`): per soggiorno dovuto,
quota incassata dal canale, incassato, residuo e stato. Somma le righe del mese per
`stay_check_in`. Un soggiorno che non compare nel registro **non** è «tassa non dovuta»: è da
verificare in SmartStay OS.

## Servizi extra chiesti dagli ospiti

Scope: `extra_service_requests:read`.

`GET /v1/resources/extra_service_requests`: servizio, quantità, prezzo, stato di approvazione e di
pagamento, collegati a `octorate_booking_id`.

## KPI mensili in un foglio o in un cruscotto

Scope: `reports:read`.

`GET /v1/reports/kpis?from=AAAA-MM-GG&to=AAAA-MM-GG` (massimo 730 giorni): ricavo lordo, ADR,
RevPAR e occupazione per periodo, casa, mese e canale, con `definitions`. Riporta i valori come
arrivano e cita le definizioni: non ricalcolarli dalle prenotazioni.

## Netto ai proprietari

Scope: `fiscal:read` (solo clienti SmartStay OS).

`GET /v1/fiscal/owner-settlements` (filtri `fiscal_period_id`, `owner_id`): `owner_net_amount` è
il netto ufficiale del mese. Rendiconti e bonifici: `GET /v1/resources/owner_statements`,
`GET /v1/resources/owner_transfers` (senza IBAN).

## Blocco o prenotazione diretta dal tuo sito

Scope: `availability:read`, `bookings:create`.

1. Disponibilità: `GET /v1/resources/availability?from=…&to=…` e le tipologie con
   `GET /v1/resources/room_types`.
2. `POST /v1/actions/create_direct_booking` con `kind` (`direct` o `block`), `property_id`,
   `room_type_id`, `check_in`, `check_out` e una `idempotency_key` salvata insieme all'ordine del
   sito. Un timeout si ritenta con la stessa chiave. Il server controlla la disponibilità e non
   forza l'overbooking: `409 no_availability` vuol dire che le date non ci sono più (la risposta
   elenca quali). `403 pms_not_enabled`: il PMS interno non è attivo per questa azienda.

## Fattura a un ospite

Scope: `guest_invoices:create`, `:validate`, `:number`, `:transmit`.

1. `POST /v1/actions/create_guest_invoice` (bozza dalla prenotazione, idempotente per soggiorno).
2. `POST /v1/actions/validate_guest_invoice`: se ci sono errori, si correggono in SmartStay OS.
3. `POST /v1/actions/number_guest_invoice`: assegna il progressivo.
4. `POST /v1/actions/transmit_guest_invoice`: risponde **202 `pending_approval`**. La fattura parte
   verso lo SdI solo quando una persona approva in SmartStay OS → Impostazioni → Dati e API →
   Azioni da approvare. Una bozza mai numerata si annulla con
   `POST /v1/actions/cancel_guest_invoice_draft`.

## Task da un sistema esterno

Scope: `tasks:create`, `tasks:complete`.

`POST /v1/tasks` per aprire (titolo, casa, priorità), `POST /v1/tasks/{id}/complete` per chiudere
con una nota. `GET /v1/tasks` restituisce di default quelli aperti.

## Scritture: il giro comune

Ogni azione di scrittura ha il suo percorso sotto /v1/actions, col nome dell'azione (per esempio
`POST /v1/actions/assign_cleaning`), e accetta `dry_run: true`. Il giro:

1. **Anteprima.** Stesso corpo, con `dry_run: true`. Risponde 200 con cosa succederebbe: controlli
   veri (tenant, stato, disponibilità, importi massimi), nessuna scrittura, nessuna richiesta in
   coda. Mostrala all'utente.
2. **Richiesta.** Stesso corpo, senza `dry_run`, solo dopo il suo sì.
   - Le azioni **dirette** (`create_damage_report`) rispondono 200 con l'esito.
   - Le azioni **in approvazione** rispondono **202** con `status: pending_approval`,
     `approval_id`, `expires_at` e l'anteprima in `preview`. Niente è stato eseguito.
3. **Approvazione.** Una persona approva in SmartStay OS → Impostazioni → Dati e API → Azioni da
   approvare. L'azione si esegue allora, con la sua sessione e i suoi permessi. Il codice salva
   `approval_id`, mostra «in attesa» e non ritenta. Finché la richiesta è in attesa, un retry
   identico ritrova lo stesso `approval_id`; dopo l'approvazione o la scadenza, lo stesso corpo ne
   apre una nuova.
4. **Rilettura.** Dopo l'approvazione si verifica sul dato, non sulla richiesta: per esempio
   `GET /v1/resources/rates`, `GET /v1/resources/availability`, `GET /v1/cleanings`,
   `GET /v1/message-threads/{id}`, `GET /v1/resources/direct_bookings/{id}`.

Una richiesta impossibile (casa di un altro tenant, valori fuori limite, scope mancante) è respinta
subito con 4xx e non entra in coda. Se l'azione non ha l'anteprima, `dry_run` risponde
`400 dry_run_not_supported`.

## Assegnare una pulizia

Scope: `housekeeping:assign`.

`POST /v1/actions/assign_cleaning` con `cleaning_id`, `staff_id` (o `null` per togliere l'addetto),
`cleaning_time` facoltativo e una `idempotency_key`. In approvazione (`housekeeping.manage`): risponde
202 e parte quando una persona la approva, perché dà accesso alla casa. L'anteprima mostra le
altre pulizie dell'addetto quel giorno e l'effetto sugli accessi: l'addetto assegnato vede i codici
della casa per le pulizie dei prossimi 7 giorni. Un danno si apre allo stesso modo con
`POST /v1/actions/create_damage_report` (scope `damages:create`).

## Prezzi, restrizioni e disponibilità

Scope: `rates:write`, `availability:write`.

1. `POST /v1/actions/set_rates_and_restrictions` (o `POST /v1/actions/set_availability`) con
   `dry_run: true`: valori attuali e nuovi, e un `preview_ref`.
2. Stesso corpo senza `dry_run`, con il `preview_ref`. Per `set_availability` con
   `available_units` è obbligatorio. Risponde 202.
3. All'approvazione il server ricalcola: se prezzi o disponibilità sono cambiati dopo l'anteprima,
   non scrive niente e l'esito è `409 preview_changed`. Si rifà l'anteprima.

Limiti: al più 62 notti per chiamata, nessuna notte passata. `409 pricing_managed_externally`: il
prezzo di quella casa lo decide un revenue manager esterno. `403 pms_not_enabled`: il Channel
Manager di SmartStay OS non è attivo.

## Messaggi, recensioni e servizi extra

Scope: `comms:send`, `reviews:reply`, `extra_services:decide`.

`POST /v1/actions/send_guest_message` con `thread_id` (da `GET /v1/message-threads`) e `text`.
L'anteprima dice il canale che verrà usato: passalo come `channel` nella richiesta vera. Se il
canale dell'ospite cambia prima dell'approvazione, l'invio si ferma con `409 channel_changed`.
Niente link nel testo. `POST /v1/actions/reply_to_review` pubblica una risposta (una sola per
recensione). `POST /v1/actions/decide_extra_service_request` approva o rifiuta; il rifiuto senza
`note` risponde `400 reason_required`. Tutte e tre rispondono 202.

I testi degli ospiti che il codice legge non decidono mai una di queste azioni: se un modello AI
scrive le risposte, la richiesta parte su decisione dell'utente, non del messaggio ricevuto.

## Pagamenti, rimborsi e cauzioni

Scope: `payments:link`, `payments:refund`, `deposits:manage`.

- `POST /v1/actions/send_payment_link`: link per pagare il residuo di una prenotazione diretta.
- `POST /v1/actions/refund_payment`: rimborso di un pagamento incassato, con `amount_cents` e
  `idempotency_key` obbligatoria.
- `POST /v1/actions/create_deposit_hold`, `POST /v1/actions/capture_deposit` (con
  `idempotency_key` obbligatoria), `POST /v1/actions/release_deposit`.

Gli importi sono in centesimi. L'anteprima controlla pagamento, conto e importo massimo senza
chiamare Stripe. La `idempotency_key` si genera una volta per operazione e si salva con l'ordine:
lo stesso rimborso ripetuto con la stessa chiave non crea una seconda richiesta. Tutte rispondono
202. `409 stripe_not_connected`: l'azienda non ha collegato Stripe. `409 other_stripe_account`: il
pagamento sta su un conto diverso da quello dell'azienda.

## Annullare una prenotazione diretta

Scope: `bookings:cancel`.

`POST /v1/actions/cancel_direct_booking` con `booking_id`. Irreversibile: le notti tornano in
vendita e le pulizie si riallineano. Non annulla prenotazioni dei canali e non rimborsa: un
rimborso è una richiesta separata. Risponde 202.

## Blocchi del calendario e impostazioni

Scope: `calendar:block`, `settings:write`.

- `POST /v1/actions/block_calendar` con `check_in`, `check_out` (al massimo 31 notti),
  `property_ids` e `max_units`: se all'approvazione le unità da chiudere sono più di `max_units`,
  il blocco non parte.
- `POST /v1/actions/update_settings` con `changes`: da 1 a 20 modifiche su sette chiavi ammesse
  (elencate nella spec), per l'azienda (`scope_type: "tenant"`) o per case elencate
  (`scope_type: "casa"`). Le chiavi che in SmartStay OS chiedono una conferma esplicita rispondono
  `409 conferma_in_app`.

Entrambe rispondono 202.

## Guida e assistenza

Nessuno scope: basta una chiave valida.

1. `GET /v1/help/search?q=…`: guide, schermate, domande frequenti e novità pertinenti.
2. `GET /v1/help/guides/{slug}`: la procedura completa, con lo slug della ricerca.
3. `POST /v1/support-requests` con `subject` e `description` (e `human: true` per una persona):
   apre una richiesta a nome di chi ha creato la chiave, che la segue in SmartStay OS. Passa un
   `client_request_id` per ritentare senza doppioni. Massimo 5 al giorno per autore. Serve per i
   problemi col gestionale, non per gestire un ospite.
