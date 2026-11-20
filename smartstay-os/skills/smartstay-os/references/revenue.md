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

Due tool cambiano prezzi e disponibilità sul Channel Manager di SmartStay OS. Sono **in
approvazione**: rispondono `pending_approval` e partono solo quando una persona approva.

- `set_rates_and_restrictions`: prezzo (fisso, variazione percentuale o in valuta) e restrizioni
  (soggiorno minimo e massimo, chiuso all'arrivo, chiuso alla partenza, stop vendita) di una casa,
  per un periodo e i giorni della settimana scelti. Senza `rate_plan_ids` vale per tutti i piani
  base.
- `set_availability`: unità disponibili di un tipo di camera, come valore assoluto
  (`available_units`) o come variazione (`change`).

Limiti: al più 62 notti per chiamata, nessuna notte già passata (ora di Roma). Le case col prezzo
affidato a un revenue manager esterno non si toccano (`pricing_managed_externally`). Serve il PMS
interno attivo (`pms_not_enabled`).

Giro:

1. Chiama il tool con `dry_run: true`. L'anteprima mostra i valori attuali e quelli nuovi, notte per
   notte, e restituisce un `preview_ref`.
2. Mostra all'utente cosa cambia: casa, periodo, giorni, prima e dopo. Chiedi il suo sì esplicito.
3. Chiama senza `dry_run`, passando il `preview_ref`. Con `available_units` è obbligatorio.
4. La risposta è `pending_approval`: niente è cambiato. Di' all'utente che va approvato in SmartStay
   OS → Impostazioni → Dati e API → Azioni da approvare. Se all'approvazione i valori non sono più
   quelli dell'anteprima, non cambia niente (`preview_changed`): si rifà l'anteprima.

Dopo l'approvazione l'invio ai canali parte da solo. Proponi una variazione solo con i dati che la
giustificano (vedi Metodo), mai come «sicuramente» redditizia.
