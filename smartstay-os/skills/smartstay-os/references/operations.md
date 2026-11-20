# Operatività quotidiana

Per: arrivi e partenze, check-in da completare, pulizie, task aperti, priorità del giorno,
pianificazione dei prossimi giorni.

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

## Confronti nel tempo

Su richiesta, confronta periodi per vedere accumulo di task, problemi ricorrenti e carichi di
lavoro. Distingui i problemi reali dai rischi possibili, e segnala ciò che si potrebbe
automatizzare (vedi `automation-advisor.md`).

## Limiti

- Nessun messaggio agli ospiti parte da qui: prepari bozze (vedi `guest-experience.md`).
- Non modificare task o prenotazioni senza conferma.
