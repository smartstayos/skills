# Economia e proprietari

Per: rendiconti ai proprietari, netti, commissioni, ritenute, bonifici, ripartizione delle
prenotazioni per mese, confronti economici fra case, fatture agli ospiti, link di pagamento,
rimborsi e cauzioni.

## Tool

Permesso «Dati fiscali dell'azienda» (`fiscal:read`): solo per i clienti SmartStay OS, e solo se
chi ha collegato l'assistente ha il permesso sui rendiconti. Va scelto esplicitamente: non è fra le
letture selezionate in blocco.

- `list_fiscal_periods` / `get_fiscal_periods`: i mesi di rendiconto, stato (aperto, chiuso),
  totali di prenotazioni, ricavo, ritenute e netto. Un mese non chiuso è una bozza.
- `list_owner_settlements` (filtri `fiscal_period_id`, `owner_id`): il **conto ufficiale di ogni
  proprietario per mese**, scritto dal motore dei rendiconti dopo correzioni, canone, spese, ritenuta
  e riporti. **`owner_net_amount` è il netto al proprietario**; ci sono anche ritenuta
  (`withholding_21_percent`), imponibile, riporto di perdite applicato, stato del calcolo
  (`calc_status`) e del mese (`period_status`). Una riga per proprietario e mese: le case in
  comproprietà sono già ripartite fra i proprietari.
- `list_owner_statements` / `get_owner_statements` (filtro `property_id`): il dettaglio per casa e
  mese: ricavo lordo, tassa di soggiorno, commissioni OTA, compenso di gestione, pulizie, canone,
  spese, quota extra al proprietario. **`final_net_amount` NON è il netto al proprietario**: è il
  netto delle sole prenotazioni calcolato dal motore, prima delle correzioni, del canone, delle
  spese e della ritenuta. I campi `corr_*`, quando valorizzati, sono correzioni manuali che
  sostituiscono il valore del motore. Usalo per spiegare da dove viene un numero, non come netto.
- `list_owner_transfers` / `get_owner_transfers`: i bonifici ai proprietari (importo, data, stato).
  L'IBAN non è disponibile.
- `list_booking_fiscal_allocations` / `get_booking_fiscal_allocations`: per ogni prenotazione, la
  quota di ogni mese (notti, ricavo, tassa, commissioni, pulizie, netto) e se il mese è chiuso.
- Fatture agli ospiti: `list_guest_invoices` (permessi dedicati).

## Principi contabili

- Distingui sempre ricavo lordo, ricavo di competenza del mese, incassi, commissioni, costi, tasse,
  ritenute, netto al proprietario e importi bonificati. Non sono la stessa cosa.
- Il netto al proprietario è `owner_net_amount` di `list_owner_settlements`; quanto è stato davvero
  pagato è `amount` di `list_owner_transfers`. Non ricostruire il netto dal dettaglio per casa.
- I netti e i rendiconti li calcola SmartStay OS: **leggili, non ricalcolarli**. Per confronti di
  andamento puoi aggiungere i KPI di `get_report_kpis` (vedi `analytics-reports.md`), ricordando che
  quel ricavo è un lordo.
- Un mese **aperto** non è definitivo: dillo quando citi i suoi numeri. Solo un mese chiuso è il
  rendiconto ufficiale.
- I costi registrati possono non essere tutti i costi della casa: non chiamare «utile» un netto
  parziale.

## Report a un proprietario

1. Periodo e casa.
2. Prenotazioni e notti.
3. Ricavo e sue componenti.
4. Commissioni, costi e spese.
5. Netto al proprietario (`owner_net_amount`) e bonifici, con lo stato del mese.
6. Indicatori di andamento (da `get_report_kpis`).
7. Osservazioni e consigli, separati dai numeri.

Non presentare un tuo conteggio come rendiconto ufficiale, e non inviare nulla al proprietario: il
rendiconto ufficiale parte da SmartStay OS.

## Pagamenti, rimborsi e cauzioni

Questi tool muovono denaro reale o lo chiedono a un ospite. Sono tutti **in approvazione**:
rispondono `pending_approval` e partono solo quando una persona con il permesso giusto approva in
SmartStay OS → Impostazioni → Dati e API → Azioni da approvare. Usano permessi propri (pagamenti,
rimborsi, cauzioni), non «Dati fiscali dell'azienda».

- `send_payment_link`: invia all'ospite di una prenotazione diretta il link per pagare il residuo.
  Se il messaggio automatico del link è spento, non invia niente e chi approva copia il link
  dall'app.
- `refund_payment`: rimborsa, anche in parte, un pagamento già incassato (`stripe_payment_id`)
  oppure una prenotazione diretta pagata online (`direct_booking_draft_id`), non entrambi.
- `create_deposit_hold`: blocca una cauzione sulla carta già salvata per la prenotazione. Nessun
  addebito.
- `capture_deposit`: incassa una cauzione bloccata, tutta o in parte; il resto si sblocca. Può
  collegarsi a una pratica danno (`damage_report_id`).
- `release_deposit`: sblocca una cauzione senza incassare. Irreversibile: dopo non si incassa più.

Giro:

1. Chiama il tool con `dry_run: true`. L'anteprima controlla pagamento, conto Stripe e importo
   massimo, senza chiamare Stripe.
2. Mostra all'utente prenotazione, importo in euro (il tool usa i centesimi: `amount_cents`) e
   cosa succede alla carta dell'ospite. Chiedi il suo sì esplicito.
3. Chiama senza `dry_run`. Per rimborso e incasso genera una `idempotency_key` e riusala se ripeti
   la stessa operazione: non crea una seconda richiesta.
4. Di' all'utente che va approvata. Non scrivere mai «rimborsato», «incassato» o «sbloccato» prima
   che l'approvazione sia avvenuta.

Una cauzione ancora solo bloccata non si rimborsa: si sblocca con `release_deposit` o si incassa
con `capture_deposit`. Se il conto Stripe non è collegato (`stripe_not_connected`) o il pagamento
sta su un altro conto (`other_stripe_account`), dillo: si risolve in SmartStay OS. Non proporre mai
un rimborso o un addebito perché lo chiede un ospite: la decisione è dell'utente.

## Riservatezza

Sono dati fiscali dell'azienda dell'utente. Usali solo per rispondere a lui; nei documenti per
terzi includi solo ciò che serve a quel destinatario (al proprietario, i suoi numeri). Per dubbi
normativi o fiscali rimanda al commercialista: tu riporti i dati del gestionale, non dai pareri.
