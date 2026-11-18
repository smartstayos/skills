# SmartStay OS per Claude

Plugin ufficiale di [SmartStay OS](https://smartstayos.com), il gestionale per gli affitti brevi.
Collega Claude al tuo account SmartStay OS e gli insegna a usarlo: prenotazioni, case, check-in,
pulizie, ospiti, report, rendiconti, la guida del gestionale e le richieste di assistenza.

## Installazione (Claude Code, Cowork)

```
/plugin marketplace add smartstayos/skills
```

```
/plugin install smartstay-os@smartstay-os
```

Al primo uso Claude apre l'accesso SmartStay OS: entri con il tuo account e scegli quali permessi
concedere. Non c'è nessun token da copiare.

Usi Claude.ai, l'app desktop o un altro client MCP senza plugin? Aggiungi il connettore
`https://api.smartstayos.com/mcp`: istruzioni e flussi pronti arrivano direttamente dal server.

## Cosa contiene

| Pezzo | Cosa fa |
|---|---|
| `smartstay-os/.mcp.json` | Il connettore MCP di SmartStay OS, con accesso OAuth. |
| `smartstay-os/skills/smartstay-os/` | La skill «SmartStay OS Agent Skills»: quando usare SmartStay OS, quali strumenti esistono, come chiedere conferma, sicurezza, più nove moduli caricati solo quando servono (operazioni, prenotazioni, pulizie, ospiti, automazioni, guida e assistenza, report, revenue, economia e proprietari). |
| `smartstay-os/commands/` | `/smartstay-os:briefing`, `/smartstay-os:aiuto`, `/smartstay-os:segnala`. |

## Privacy e permessi

- Claude vede solo i dati della tua azienda e solo con i permessi che concedi al collegamento.
- I dati fiscali (rendiconti, bonifici) richiedono un permesso dedicato, da scegliere esplicitamente.
- Le azioni delicate (personale, trasmissione fatture) non partono dal connettore: restano in attesa
  finché un amministratore non le approva in SmartStay OS.
- Revoca in qualsiasi momento: SmartStay OS → Impostazioni → Dati e API → Assistenti AI (MCP).

## Supporto

Scrivi dall'assistenza di SmartStay OS: https://smartstayos.com/os/assistenza

---

**English.** Official Claude plugin for SmartStay OS, a property management system for short-term
rentals. Install with `/plugin marketplace add smartstayos/skills` and
`/plugin install smartstay-os@smartstay-os`; sign in with your SmartStay OS account on first use.
The content is in Italian.

_Questo repository è pubblicato automaticamente: le modifiche fatte qui verrebbero sovrascritte._
