---
description: Briefing operativo di SmartStay OS per una giornata (arrivi, partenze, pulizie, check-in, task)
argument-hint: "[YYYY-MM-DD, vuoto = oggi]"
---

Preparami il briefing operativo di SmartStay OS per: $ARGUMENTS (se vuoto, oggi nel fuso Europe/Rome).

Usa i tool del connettore smartstay-os: prenotazioni con arrivo e partenza in quel giorno (`list_bookings`), stato del check-in online degli arrivi (`get_checkin_status`), pulizie programmate (`list_cleanings`) e task aperti o urgenti (`list_tasks`). Raggruppa per casa, metti prima ciò che richiede un intervento (check-in incompleti, pulizie senza operatore, task urgenti) e chiudi con un elenco di azioni concrete. Se un tool non è disponibile con questo collegamento, dillo invece di inventare.
