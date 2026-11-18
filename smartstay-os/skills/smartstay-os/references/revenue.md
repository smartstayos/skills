# Revenue

Per: prezzi, ADR, RevPAR, occupazione, ricavi, pickup, soggiorni minimi, periodi deboli,
strategie per le prossime settimane. Non sostituisce il motore tariffario del gestionale: lo aiuta a
decidere.

## Tool

- `get_report_kpis`: ADR, RevPAR e occupazione in `totals`; ADR e occupazione per casa; ricavo per
  mese e canale; `on_the_books` (arrivi futuri, quindi chiedi un periodo che arriva nel futuro),
  `pickup` (prenotazioni degli ultimi 7 e 30 giorni), `lead_time`. Vedi `definitions.sections`.
- `list_rates`, `list_rate_plans` (prezzi e restrizioni impostati; finestra massimo 90 giorni).
- `list_availability` (finestra massimo 90 giorni), `list_room_types`.
- `list_bookings` per il dettaglio di una casa o di un periodo.

## Metodo

1. **Osserva**: KPI del periodo e del periodo comparabile (stessa lunghezza, anno prima o periodo
   precedente), prenotato futuro (con un periodo da oggi in avanti) e pickup, prezzi e restrizioni impostati, notti libere.
2. **Interpreta**: occupazione, prezzo medio, buchi di calendario, soggiorni minimi che bloccano
   notti singole, canali. Non attribuire tutto al prezzo: guarda anche stagionalità, disponibilità,
   blocchi e cancellazioni.
3. **Raccomanda**: per ogni proposta casa e periodo, problema osservato, evidenza, intervento,
   beneficio atteso **come ipotesi**, rischi, verifica successiva.

## Principi

- Usa ADR, RevPAR e occupazione come li dà `get_report_kpis`, con le loro definizioni.
- Non inventare prezzi dei concorrenti, eventi locali, dati di mercato o previsioni: il gestionale
  non li ha. Se li usi da altre fonti, dillo e citale.
- Una variazione di prezzo non è mai «sicuramente» redditizia.

## Azioni

Dal connettore non si cambiano tariffe né restrizioni. Prepara la proposta e guida l'utente a
farla in SmartStay OS (procedura con `search_help_guide`).
