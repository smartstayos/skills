# Report e analisi

Per: report personalizzati, dashboard, grafici, confronti fra case o periodi, KPI, tendenze,
anomalie, report per proprietari o collaboratori, esportazioni.

## Tool

- `get_report_kpis` (`from`, `to`, massimo 730 giorni; permesso «Report»): gli indicatori
  ufficiali di SmartStay OS per le **prenotazioni con arrivo nel periodo**, con le definizioni
  principali in `definitions` (leggi `definitions.scope` e `definitions.sections`):
  - `totals`: prenotazioni, notti, ricavo (`revenue`), ADR, RevPAR, occupazione, notti disponibili,
    durata e ospiti medi, cancellazioni;
  - `per_property`: prenotazioni, notti, ricavo, ADR, occupazione, costi (niente RevPAR);
  - `monthly`: prenotazioni, notti, ricavo, ADR (niente occupazione né RevPAR);
  - `per_channel`: prenotazioni, ricavo, quota;
  - `on_the_books` (solo arrivi futuri dentro il periodo chiesto: vuoto per un periodo passato),
    `pickup` (ultimi 7 e 30 giorni, indipendente dal periodo), `lead_time`, `cost_incidence`,
    `blocks` (notti tolte dalla vendita), `reviews` (gli estratti solo con il permesso recensioni),
    `tickets`.
  Le notti e l'importo di un soggiorno vanno interi al mese d'arrivo: per la competenza per mese
  servono i dati fiscali (`list_booking_fiscal_allocations`).
- Per il dettaglio: `list_bookings`, `get_booking`, `list_properties`.
- Per i netti e i rendiconti ufficiali: vedi `finance-owners.md`.

## Principio

SmartStay OS dà numeri e definizioni; tu costruisci l'analisi. Usa i KPI di `get_report_kpis` così
come arrivano: **non ricalcolare** ADR, RevPAR, occupazione o ricavo con una formula tua, e non
mescolarli con conteggi fatti a mano sulle prenotazioni (le regole su cancellazioni, no-show e
blocchi sono diverse). Se ti serve una metrica che non c'è, dillo e spiega come l'hai ricavata.

`revenue` è un **lordo maturato** (soggiorni realizzati più penali): non è un netto né un incasso.
Per il netto al proprietario si usano i rendiconti.

## Procedura

1. **Domanda**: obiettivo, periodo, case, metriche, confronti, formato. Se manca qualcosa scegli un
   valore ragionevole e dichiaralo; chiedi solo se cambierebbe il risultato.
2. **Dati**: una chiamata a `get_report_kpis` per periodo. Per un confronto (per esempio con lo
   stesso periodo dell'anno prima) fai due chiamate con periodi della stessa lunghezza.
3. **Verifica**: periodo, case incluse, notti disponibili (se `available_nights` è 0, occupazione e
   RevPAR non sono calcolabili: dillo, non scrivere 0%). Un dato assente non è uno zero.
4. **Analisi**: variazioni, case sopra e sotto la media, canali, stagionalità, anomalie. Niente
   conclusioni causali che i dati non reggono.
5. **Presentazione**:
   1. obiettivo e ambito (periodo, case, filtri);
   2. sintesi in tre righe;
   3. KPI principali;
   4. confronti, tabelle e grafici se il client li supporta;
   5. anomalie;
   6. interpretazione, separata dai fatti;
   7. raccomandazioni con priorità;
   8. limiti dei dati.

   Per una domanda semplice basta la risposta, non un report.

## Raccomandazioni

Ognuna con: evidenza, interpretazione, azione proposta, priorità, indicatore da monitorare.

## Report ricorrenti

Puoi proporre una struttura riutilizzabile. Non promettere invii automatici futuri: il connettore
non pianifica report.

## Limiti

- Non dichiarare creato un file che non hai prodotto.
- Nei report per terzi riduci i dati personali degli ospiti al necessario.
