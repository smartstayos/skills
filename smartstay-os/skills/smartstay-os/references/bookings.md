# Prenotazioni, disponibilità e case

Per: prenotazioni passate, presenti e future, disponibilità, calendario, durata dei soggiorni,
cancellazioni, anticipo di prenotazione, notti libere, confronti fra case.

## Tool

- `list_properties`, `get_property` (dettaglio operativo; codici di accesso e password Wi-Fi non
  sono mai restituiti).
- `list_bookings`, `get_booking`, `search` / `fetch` (ricerca per nome ospite, casa o task).
- `list_availability` (finestra `from`/`to` obbligatoria, massimo 90 giorni), `list_room_types`,
  `list_property_units`.
- `list_rate_plans`, `list_rates` (finestra obbligatoria): piani tariffari e prezzi impostati.
- `list_direct_bookings`: prenotazioni dirette.
- Scritture, solo con il permesso e dopo conferma: `create_direct_booking`,
  `update_direct_booking`.

## Procedura

1. Fissa periodo e case. Per la disponibilità rispetta il limite di 90 giorni per chiamata.
2. Leggi tutte le pagine dei risultati prima di contare.
3. Non confondere prenotazioni confermate, cancellate, richieste, blocchi del calendario e
   soggiorni conclusi. Una notte bloccata non è una notte vendibile.

## Cosa puoi analizzare

- **Notti e soggiorni**: notti prenotate per casa, durata media e distribuzione.
- **Anticipo**: tempo tra la data di prenotazione e l'arrivo, se il dato c'è.
- **Cancellazioni**: quante, quando, con quale effetto sul calendario.
- **Notti libere**: buchi fra un soggiorno e l'altro, cambi stretti (partenza e arrivo lo stesso
  giorno).
- **Confronti** fra case o periodi equivalenti, segnalando stagionalità e differenze di
  disponibilità.

Per occupazione, ADR, RevPAR e ricavi usa `get_report_kpis` (vedi `analytics-reports.md`), che
ha le regole ufficiali su cancellazioni, no-show e blocchi. Se conti tu qualcosa sulle prenotazioni
lette, dichiara la regola usata e non presentarlo come dato di SmartStay OS.

## Consigli

Puoi suggerire di rivedere soggiorni minimi, gestione dei buchi, disponibilità o cancellazioni,
dicendo sempre su quali dati ti basi. Le modifiche a tariffe e restrizioni si fanno in SmartStay OS
(cerca la procedura con `search_help_guide`).

## Limiti

- Non modificare disponibilità, restrizioni o prenotazioni senza conferma e permesso.
- Senza lo storico necessario, non inventare confronti.
