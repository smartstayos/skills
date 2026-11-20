# Autenticazione e permessi

## La chiave

- Si crea in SmartStay OS → Impostazioni → Dati e API → API & integrazioni. La crea chi ha il
  permesso di gestire le integrazioni; una chiave non può ricevere permessi che il suo creatore non ha.
- Formato `ssk_` seguito da 64 caratteri esadecimali. Si vede **una volta sola**: va salvata subito
  nel secret manager o nel file d'ambiente dell'utente (mai committato).
- Si passa a ogni richiesta come `Authorization: Bearer ssk_…` oppure `X-API-Key: ssk_…`.
- Scadenza facoltativa (da 1 a 730 giorni), revoca immediata dalla stessa schermata, al massimo 10
  chiavi attive per azienda.
- A ogni chiamata il server ricontrolla anche i permessi **attuali** del creatore: se la persona
  lascia l'azienda o perde un permesso, la chiave smette di funzionare (403 `forbidden`). Per
  un'integrazione che deve durare, la chiave la crea un amministratore stabile.

## Rotazione senza interruzioni

1. Crea la chiave nuova con gli stessi scope.
2. Aggiorna il secret dell'integrazione e riavviala.
3. Controlla con `GET /v1/me` che risponda con la chiave nuova.
4. Revoca la vecchia.

## Quali scope chiedere

L'elenco completo, con la descrizione, è nella pagina `/docs` (sezione Scope) e in ogni operazione
della spec (`x-required-scope`). I più usati:

| Bisogno | Scope |
|---|---|
| case e unità | `properties:read` |
| prenotazioni | `bookings:read` (+ `operations:read` per lo stato operativo completo) |
| stato del check-in online | `checkin:read` |
| pulizie | `housekeeping:read` |
| task | `tasks:read`, `tasks:create`, `tasks:update`, `tasks:complete` |
| tassa di soggiorno | `tourist_tax:read` (solo clienti SmartStay OS) |
| servizi extra chiesti dagli ospiti | `extra_service_requests:read` |
| disponibilità e tariffe | `availability:read`, `rates:read` (finestra di 90 giorni) |
| KPI ufficiali | `reports:read` |
| rendiconti, netto al proprietario, bonifici | `fiscal:read` (solo clienti SmartStay OS, va scelto a mano) |
| prenotazioni dirette e blocchi | `bookings:create`, `bookings:update` |
| fatture ospiti | `guest_invoices:read`, `:create`, `:validate`, `:number`, `:transmit` |

Scope di scrittura con approvazione: la richiesta risponde `202 pending_approval` e parte solo
quando una persona con il permesso corrispondente la approva in SmartStay OS → Impostazioni → Dati
e API → Azioni da approvare. Si esegue con la sessione di chi approva, non con la chiave.

| Bisogno | Scope |
|---|---|
| prezzi e restrizioni sui canali | `rates:write` |
| disponibilità sui canali | `availability:write` |
| blocchi del calendario | `calendar:block` |
| sette regole della casa (silenzio, fuso, politiche, contatti) | `settings:write` |
| messaggi agli ospiti | `comms:send` |
| risposte alle recensioni | `reviews:reply` |
| decisioni sui servizi extra | `extra_services:decide` |
| link per pagare il residuo | `payments:link` |
| rimborsi | `payments:refund` |
| cauzioni: blocco, incasso, sblocco | `deposits:manage` |
| annullamento di una prenotazione diretta | `bookings:cancel` |

Scope di scrittura diretti, senza approvazione:

| Bisogno | Scope |
|---|---|
| assegnare una pulizia a un addetto | `housekeeping:assign` |
| aprire una pratica danno | `damages:create` |

Guida (`GET /v1/help/search`, `GET /v1/help/guides/{slug}`) e assistenza
(`POST /v1/support-requests`) non chiedono scope: basta una chiave valida. La richiesta di
assistenza parte a nome di chi ha creato la chiave.

Regola: leggere non implica mai scrivere, e non esistono scope «jolly». Uno scope di scrittura
non basta da solo: anche chi crea la chiave deve avere il permesso corrispondente, e per le azioni
in approvazione deve averlo anche chi approva. Se una chiamata risponde
403 `insufficient_scope`, manca proprio quello scope: si aggiunge creando una chiave nuova.

## Dove gira il codice

- Un backend, una funzione serverless, uno script schedulato: sì, con la chiave in variabile
  d'ambiente.
- Un'app nel browser o mobile: **no** con la chiave dentro. Passa da un proprio backend che fa da
  intermediario e che espone solo ciò che serve.
