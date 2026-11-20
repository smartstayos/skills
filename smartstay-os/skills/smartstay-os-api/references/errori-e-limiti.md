# Errori e limiti

Ogni errore ha la stessa forma:

```json
{ "error": { "code": "insufficient_scope", "message": "…" }, "request_id": "…" }
```

`request_id` c'è anche nell'header `X-Request-Id` di ogni risposta: va registrato con l'errore e
citato quando l'utente chiede assistenza a SmartStay.

| HTTP | `code` | Cosa vuol dire | Cosa fa il codice |
|---|---|---|---|
| 202 | — (`status: pending_approval`) | azione sensibile ricevuta, in attesa di approvazione (con l'anteprima in `preview`) | mostra «in attesa», salva `approval_id`, non ritenta |
| 400 | `invalid_params`, `invalid_json` e codici specifici dell'azione (`invalid_dates`, `too_many_nights`, `invalid_note`…) | parametri o corpo non validi: il messaggio dice quale campo | correggi la richiesta; non ritentare uguale |
| 400 | `dry_run_not_supported` | l'azione non ha l'anteprima | togli `dry_run` solo dopo aver mostrato all'utente cosa farà |
| 400 | `reason_required` | rifiuto di un servizio extra senza motivo | aggiungi `note` (da 3 a 1000 caratteri) |
| 401 | `unauthorized` | chiave mancante, sbagliata, revocata o scaduta | ferma l'integrazione e avvisa: serve una chiave valida |
| 403 | `insufficient_scope` | la chiave non ha lo scope dell'operazione | serve una chiave con quello scope |
| 403 | `forbidden` | il creatore della chiave non ha più il permesso, o il dato non è concesso a questa azienda | ricrea la chiave da un amministratore attuale |
| 403 | `plan_not_included`, `addon_required`, `pms_not_enabled` | il piano o l'add-on dell'azienda non comprende la funzione (per esempio CRM ospiti, PMS interno e Channel Manager) | non è un problema di chiave: va attivata la funzione in SmartStay OS |
| 404 | `not_found`, `unknown_route`, `unknown_action` | risorsa che nel tuo tenant non c'è, o percorso o azione sbagliati | controlla id e percorso nella spec |
| 405 | `method_not_allowed` | il percorso esiste con un altro metodo | correggi il metodo |
| 409 | `conflict`, `no_availability`, `movement_rejected`, `period_closed` e altri codici di stato | l'operazione non è ammessa ora (email già presente, date non disponibili, mese chiuso) | mostra il motivo all'utente |
| 409 | `preview_changed` | all'approvazione prezzi o disponibilità non erano più quelli dell'anteprima: niente è cambiato | rifai l'anteprima e una richiesta nuova |
| 409 | `channel_changed` | il canale dell'ospite non è più quello indicato in `channel` | rifai l'anteprima e usa il canale nuovo |
| 409 | `pricing_managed_externally` | il prezzo della casa lo decide un revenue manager esterno | non si scrive da qui |
| 409 | `stripe_not_connected`, `other_stripe_account` | l'azienda non ha collegato Stripe, o il pagamento sta su un altro conto | si risolve in SmartStay OS |
| 409 | `conferma_in_app` | l'impostazione chiede una conferma esplicita in SmartStay OS | va cambiata da Impostazioni, non dall'API |
| 409 | `approver_session_required` | l'azione si esegue solo all'approvazione, con la sessione di chi approva | non chiamarla come diretta: aspetta l'approvazione |
| 429 | `rate_limited` | oltre 120 richieste al minuto per chiave | attendi e ritenta (1, 2, 4, 8 s) |
| 503 | `service_unavailable` | API in pausa o momentaneamente sovraccarica | attendi e ritenta |

## Limiti

- 120 richieste al minuto per chiave. Per sincronizzazioni grandi: pagine da 100, filtri per data,
  niente richieste in parallelo oltre poche unità.
- Liste: `limit` da 1 a 100 (i messaggi di una conversazione fino a 200), `offset` da 0. Continua
  finché `pagination.has_more` è `false`. L'ordine di ogni lista è stabile (con `id` come ultimo
  criterio), quindi la paginazione per offset non salta né ripete righe tra una pagina e l'altra.
- Disponibilità e prezzi giornalieri: `from` e `to` obbligatori, massimo 90 giorni.
- KPI: periodo massimo 730 giorni.

## Ritentare o no

| Operazione | Ritento automatico |
|---|---|
| ogni `GET` | sì, su 429/502/503/504 |
| `create_direct_booking`, `add_cleaning_note`, `assign_cleaning`, `create_damage_report`, `create_staff`, `issue_staff_temporary_pin`, `record_staff_attendance` | sì, con la **stessa** `idempotency_key` |
| `commit_inventory_movement` | sì, con lo stesso `client_session_id` |
| `create_guest_invoice` | sì: è idempotente per soggiorno (rileggi la bozza esistente) |
| `POST /v1/support-requests` | sì, con lo stesso `client_request_id` |
| `POST /v1/tasks`, `POST /v1/guest-contacts`, `create_manual_guest_invoice`, `invite_guest_checkin` | **no**: prima controlla con una lettura se è già avvenuta |
| azioni in approvazione (risposta `202`) | **no**: sono già ricevute. Finché la richiesta è in attesa, un retry identico (stessa chiave, stesso corpo) ritrova lo stesso `approval_id`; dopo che è stata approvata, rifiutata o è scaduta, lo stesso corpo apre una richiesta **nuova** |
| `refund_payment`, `capture_deposit` | solo con la **stessa** `idempotency_key`, obbligatoria: senza, la richiesta è rifiutata |
| qualunque chiamata con `dry_run: true` | sì: non scrive niente |


## Compatibilità

Tutto sta sotto `/v1`. Dentro v1 possono arrivare campi, endpoint e scope nuovi, mai rimozioni o
rinomine: il codice deve ignorare i campi che non conosce, non fallire. Un cambiamento incompatibile
arriverà come `/v2`.
