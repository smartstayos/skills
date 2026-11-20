# Plugin SmartStay OS per Claude

Collega Claude (Claude Code, Cowork) a SmartStay OS in un passo e gli insegna come usarlo.

| Pezzo | Cosa fa |
|---|---|
| `.mcp.json` | Connettore MCP `https://api.smartstayos.com/mcp`. Al primo uso Claude apre l'accesso SmartStay (OAuth): scegli i permessi da concedere. Nessun token da copiare. |
| `skills/smartstay-os/SKILL.md` | Skill principale (v1.0): quando usare SmartStay OS, quali tool esistono davvero, cosa il connettore non ha ancora, metodo, conferme, sicurezza, memoria. |
| `skills/smartstay-os/references/` | Nove moduli caricati solo quando servono: `operations`, `bookings`, `housekeeping`, `guest-experience`, `automation-advisor`, `help-support`, `analytics-reports`, `revenue`, `finance-owners`. |
| `skills/smartstay-os-api/` | Skill per chi scrive codice (v1.0): integrare l'API REST di SmartStay OS partendo dalla spec OpenAPI, con chiavi fuori dal codice, scope minimi, paginazione, nuovi tentativi, idempotenza e `202 pending_approval`. Moduli: `autenticazione-e-permessi`, `client-di-riferimento`, `flussi-comuni`, `errori-e-limiti`. |
| `commands/` | `/smartstay-os:briefing`, `/smartstay-os:aiuto`, `/smartstay-os:segnala`. |

## Installazione

```
/plugin marketplace add smartstayos/skills
```

```
/plugin install smartstay-os@smartstay-os
```

I client che non usano plugin (ChatGPT, Claude.ai senza plugin, altri client MCP) ricevono le
stesse istruzioni e gli stessi flussi direttamente dal server: istruzioni di `initialize` e
`prompts/list`.

## Revoca

SmartStay OS → Impostazioni → Dati e API → Assistenti AI (MCP): revoca il collegamento
`OAuth: <nome app>`.

## Permessi per report e dati fiscali

I moduli `analytics-reports` e `revenue` usano il permesso «Report» (KPI ufficiali, `get_report_kpis`).
`finance-owners` usa «Dati fiscali dell'azienda» (rendiconti, bonifici, ripartizioni): solo per i
clienti SmartStay OS, solo se chi collega ha il permesso sui rendiconti, e va scelto esplicitamente.
