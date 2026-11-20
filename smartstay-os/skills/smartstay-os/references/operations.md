# Operatività quotidiana

Per: arrivi e partenze, check-in da completare, pulizie, task aperti, priorità del giorno,
pianificazione dei prossimi giorni, blocchi del calendario, regole e contatti della casa.

## Tool

- `list_bookings` (`from`, `to`, `property_id`): prenotazioni nel periodo. Le cancellate sono
  escluse salvo `include_cancelled`.
- `get_booking` con `include_operational_state: true`: stato derivato del soggiorno (check-in
  online, cauzione, tassa di soggiorno, pulizie, anomalie). Serve il permesso `operations:read`.
- `get_checkin_status`: fase del check-in online di una prenotazione.
- `list_cleanings`, `list_tasks`, `get_task`.
- `list_tourist_tax_ledger` (`property_id`): tassa di soggiorno per soggiorno, con dovuto, quota
  incassata dal canale, incassato e residuo. Permesso `tourist_tax:read`. Un soggiorno che non è
  nel registro non vuol dire «non dovuta»: dillo come «da verificare».
- Scritture, solo con il permesso e dopo conferma: `create_task`, `update_task`, `complete_task`,
  `add_cleaning_note`.
- Scritture **in approvazione** (rispondono `pending_approval`):
  - `block_calendar`: chiude le notti da `check_in` a `check_out` (al massimo 31) sulle case
    elencate e le sincronizza con i canali. `max_units` è il tetto di unità che si possono chiudere:
    se l'anteprima ufficiale ne trova di più, il blocco non parte.
  - `update_settings`: cambia sette regole della casa (orari di silenzio, fuso orario, politiche
    che l'assistente comunica agli ospiti, contatti di emergenza e manutenzione), per tutta
    l'azienda o come eccezione su case elencate. Le altre impostazioni, compresi orari di check-in
    e check-out, cauzione e accessi, si cambiano in SmartStay OS (`conferma_in_app`).

## Procedura

1. Fissa giorno e case (default: oggi, tutte).
2. Leggi arrivi e partenze del giorno. Per ogni arrivo guarda lo stato del check-in e, se serve,
   lo stato operativo.
3. Leggi pulizie e task del giorno.
4. Considera completato solo ciò che il gestionale dà come completato.

## Briefing

1. **Situazione**: arrivi, partenze, cambi nello stesso giorno, pulizie, check-in incompleti, task
   aperti e urgenti, casa per casa.
2. **Criticità**: solo problemi verificati o rischi motivati dai dati (per esempio arrivo oggi con
   check-in non completato, pulizia senza operatore prima di un arrivo).
3. **Azioni**: ordinate per urgenza, impatto sull'ospite e scadenza.

## Bloccare il calendario o cambiare una regola

1. Chiama `block_calendar` o `update_settings` con `dry_run: true` e mostra l'anteprima. Per un
   blocco: case, date e notti; le unità esatte le conta il server all'approvazione. Per una
   regola: valore attuale e nuovo, per l'azienda o per ogni casa.
2. Chiedi il sì esplicito dell'utente. Per un blocco, concorda con lui `max_units`.
3. Chiama senza `dry_run`. La risposta è `pending_approval`: niente è cambiato finché una persona non
   approva in SmartStay OS → Impostazioni → Dati e API → Azioni da approvare. Dillo così.

## Confronti nel tempo

Su richiesta, confronta periodi per vedere accumulo di task, problemi ricorrenti e carichi di
lavoro. Distingui i problemi reali dai rischi possibili, e segnala ciò che si potrebbe
automatizzare (vedi `automation-advisor.md`).

## Limiti

- Un messaggio agli ospiti parte solo dopo l'approvazione di una persona (vedi
  `guest-experience.md`).
- Non modificare task o prenotazioni senza conferma.
