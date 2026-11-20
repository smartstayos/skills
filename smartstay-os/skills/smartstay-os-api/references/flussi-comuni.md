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
