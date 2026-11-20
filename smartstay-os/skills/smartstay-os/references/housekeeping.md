# Pulizie e personale delle pulizie

Per: pulizie programmate, cambi, assegnazioni, ritardi, carichi di lavoro, danni, personale, orari
e presenze.

## Tool

- `list_cleanings` (default da oggi in avanti): tipo, addetto (`assigned_staff_id`, il nome con
  `get_staff`), «non prima di» / «pronta entro», ospiti e letti in arrivo, orari effettivi.
- Scritture leggere, dopo conferma: `add_cleaning_note` (nota per la pulizia di un arrivo),
  `update_cleaning_beds` (configurazione dei letti).
- Assegnazione, in approvazione (dà i codici della casa): `assign_cleaning` assegna una pulizia a un addetto attivo, o la lascia
  senza addetto (`staff_id: null`), e può cambiarne l'ora. La data non si sposta. Solo pulizie da
  fare, da oggi in avanti.
- Danni, diretta: `create_damage_report` apre una pratica danno su una prenotazione, con la sola
  descrizione. Valore, foto e addebito all'ospite si decidono poi in SmartStay OS.
- Personale, lettura: `list_staff`, `get_staff`, `get_staff_qualifications`, `get_staff_schedule`,
  `get_staff_attendance`, `list_staff_training_tracks`, `get_staff_training`,
  `list_staff_acknowledgments`.
- Personale, scrittura (**sensibile**, va in approvazione): `create_staff`, `deactivate_staff`,
  `update_staff_qualifications`, `issue_staff_temporary_pin`, `assign_staff_training`,
  `set_staff_schedule`, `record_staff_attendance`, `void_staff_attendance_event`.

## Procedura

1. Leggi le pulizie del periodo e, se servono, le prenotazioni collegate.
2. Usa solo gli stati che il gestionale restituisce: non inventare categorie.
3. Segnala per prime le pulizie senza operatore prima di un arrivo e i cambi stretti.

## Analisi

Con i dati disponibili: distribuzione delle pulizie per giorno e per operatore, cambi ravvicinati,
attività non assegnate, problemi ricorrenti. Non giudicare la qualità del lavoro dai soli tempi.

## Assegnare una pulizia

1. Chiama `assign_cleaning` con `dry_run: true`. L'anteprima dice addetto attuale e nuovo, orario,
   le altre pulizie dell'addetto quel giorno (con sovrapposizioni) e l'effetto sugli accessi.
2. Mostrala all'utente. Digli sempre che l'addetto assegnato vede i codici d'accesso della casa per
   le pulizie dei prossimi 7 giorni, e che quello precedente li perde.
3. Dopo il suo sì, chiama senza `dry_run`. Risponde `pending_approval`: l'assegnazione parte quando
   un amministratore la approva in SmartStay OS. Non dire «assegnata» prima.

Per un danno il giro è anteprima con `create_damage_report`, conferma, poi la pratica (diretta).
Nella descrizione metti cosa è danneggiato e dove, senza dati personali dell'ospite oltre al
necessario. Non promettere addebiti: li decide l'utente in SmartStay OS.

## Personale e approvazioni

Le azioni sul personale rispondono `pending_approval`: non sono eseguite finché un amministratore
non le approva in SmartStay OS. Dillo all'utente con queste parole. Prima di chiederle, mostra
cosa stai per fare (chi, cosa, da quando) e aspetta la conferma. Non vedi mai il PIN temporaneo.

## Limiti

Non assegnare pulizie, né modificare orari, presenze o abilitazioni, senza conferma. Una correzione delle presenze ha
sempre un motivo, che resta nel registro.
