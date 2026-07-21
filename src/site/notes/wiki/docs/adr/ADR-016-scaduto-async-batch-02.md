---
{"dg-publish":true,"permalink":"/wiki/docs/adr/adr-016-scaduto-async-batch-02/","title":"Stato SCADUTO gestito esclusivamente da BATCH-02 (asincrono, no scadenza sincrona durante acquisizione)","tags":["batch-02","scaduto","asincrono","informativa","semantica","mf66","rischio-semantica"],"dg-note-properties":{"adr":16,"title":"Stato SCADUTO gestito esclusivamente da BATCH-02 (asincrono, no scadenza sincrona durante acquisizione)","status":"accepted","date":"2026-03-02","deciders":["CSI Piemonte","Exprivia"],"supersedes":[],"superseded-by":[],"tags":["batch-02","scaduto","asincrono","informativa","semantica","mf66","rischio-semantica"],"related_wiki":["[[wiki/concepts/batch-processes\|Processi Batch — BATCH-01, BATCH-02, BATCH-03]]","[[wiki/concepts/ciclo-vita-consenso\|Ciclo di Vita del Consenso]]","[[wiki/concepts/informativa\|Informativa Consenso]]","[[wiki/analyses/analysis-gap-as-is-to-be\|Analisi Gap AS-IS → TO-BE]]"],"sources":["[[2026-03-02-conspref-srs-v1-revised|CONSPREF-SRS-V1.0 revised bozza v2]] §7.2","[[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|Risposte MF SRS v3]] MF66R65","[[2019-03-20-acc-del-cdu-01-servizi-acquisizione|ACC-DEL-CDU-01 (AS-IS)]]"]}}
---


# ADR-016: Stato SCADUTO gestito da BATCH-02 (asincrono)

## Status

`accepted` — semantica TO-BE formalizzata in SRS §7.2; SQL canonico definito da MF66R65.

## Context

Comportamento AS-IS ([[wiki/sources/2019-03-20-acc-del-cdu-01-servizi-acquisizione\|ACC-DEL-CDU-01]]): lo stato `SCADUTO` viene impostato **durante l'operazione di acquisizione** (sincrono), se al momento del rilascio il consenso esistente è collegato a un'informativa scaduta.

Limiti AS-IS:
- Logica di scadenza accoppiata all'acquisizione (CDU-03 lento, complesso)
- Non c'è transizione SCADUTO autonoma se nessuno tenta acquisizione (consensi "dimenticati" restano ATTIVO oltre la scadenza informativa)
- Mix sincronia/asincronia difficile da testare

TO-BE introduce processi batch (vedi [[wiki/concepts/batch-processes\|Processi Batch]]) per disaccoppiare:
- BATCH-01 (acquisizioni → notifiche)
- BATCH-02 (scadenza informative → transizioni SCADUTO/ANNULLATO)

## Decision

Lo stato `SCADUTO` è gestito **esclusivamente da BATCH-02**, asincrono, alla scadenza dell'informativa. CDU-03 e CDU-04 **non** impostano più `SCADUTO` durante acquisizione.

Workflow BATCH-02:
1. Trovare informative scadute (`data_scadenza < NOW()`)
2. Per ogni informativa scaduta, determinare la nuova informativa corrente con SQL canonico (MF66R65):

```sql
SELECT d_informativa_id AS nuova_d_informativa_id
FROM cons_d_informativa
WHERE sotto_tipo_consenso = :sotto_tipo_consenso_scaduta
  AND (data_scadenza IS NULL OR data_scadenza > NOW())
  AND data_decorrenza <= NOW()
  AND data_cancellazione IS NULL
ORDER BY data_decorrenza DESC
LIMIT 1;
```

3. Per ogni `cons_id` collegato all'informativa scaduta in stato `ATTIVO`/`NEGATO`:
   - **UPDATE** chiusura record corrente (`data_fine = NOW()`, `login_operazione = 'BATCH_SCADENZA_INF'`)
   - **INSERT** nuovo record storicizzato con stato `SCADUTO` (se `annulla_consensi=NO`) o `ANNULLATO` (se `annulla_consensi=SI`), puntando alla nuova informativa
4. Se `ANNULLATO` → INSERT in `cons_t_notifica` → BATCH-01 notifica SIA

Storicizzazione segue [[wiki/docs/adr/ADR-015-storicizzazione-immutabile\|ADR-015-storicizzazione-immutabile]] (immutabile).

| `annulla_consensi` | Nuovo stato | Notifica ASR? |
|---|---|---|
| NO | SCADUTO | No |
| SI | ANNULLATO | Sì |

## Consequences

### Positive
- Disaccoppiamento sincrono/asincrono: CDU-03 più semplice e veloce
- Scadenza autonoma: consensi su informative scadute transitano automaticamente, anche senza nuova acquisizione
- SQL canonico definito (MF66R65) → niente ambiguità implementativa
- Tracciatura via `login_operazione = 'BATCH_SCADENZA_INF'` rende batch riconoscibile in audit

### Negative
- **Differenza semantica con AS-IS** — i SIA che assumono "SCADUTO arriva sincrono" devono adattarsi (rischio integrazione)
- Notifica scadenza non in tempo reale: max latenza = scheduling BATCH-02 (frequenza da definire)
- SC67 aperto: "INSERT cons_t_consenso da batch — da approfondire e verificare meglio" (vedi [[wiki/concepts/batch-processes\|Processi Batch]] §ALG02)

### Neutral
- Macchina a stati documentata in [[wiki/concepts/ciclo-vita-consenso\|Ciclo di Vita del Consenso]] coerente
- Frequenza esecuzione BATCH-02 da definire con CSI (BAT-02 nei punti aperti)

## Alternatives considered

| Alternativa | Motivo scarto |
|---|---|
| Mantenere SCADUTO sincrono AS-IS | Rigettato dal cliente con introduzione dei batch; CDU-03 troppo lento e complesso |
| SCADUTO durante read | Implicito (lettura lazy) — complica query, niente tracciatura, niente notifica |
| Trigger DB-side su `cons_d_informativa` | Logica nascosta DB-side; difficile da test/version |

## Open issues

> **Aggiornamento call CSI 20/07/2026.**

- **SC67 (BAT-02)** — 🟠 aperto, **domanda riformulata a scenario** (CSI non aveva compreso la domanda astratta): *informativa A (`annulla_consensi=NO`) scade e viene sostituita da B (`annulla_consensi=SI`); un consenso ATTIVO legato ad A → **SCADUTO** (flag di A, §6.13) o **ANNULLATO** (flag di B, §7.2 SQL)? Sorgente autoritativa del flag?* Interpretazione proposta: leggere il flag dall'informativa **scaduta A**. Vedi [[wiki/concepts/batch-processes\|Processi Batch]] §ALG02.
- **BAT-03** — 🟠 aperto: CSI considera lo Stato SCADUTO "componente da gestire" → **call dedicata pending** per la gestione asincrona e la comunicazione alle ASR.
- **Frequenza BATCH-02** — ⚪ non vincolante (call 20/07/2026), da concordare in seguito.

## References

- [[wiki/concepts/batch-processes\|Processi Batch — BATCH-01, BATCH-02, BATCH-03]] §BATCH-02
- [[wiki/concepts/ciclo-vita-consenso\|Ciclo di Vita del Consenso]]
- [[wiki/concepts/informativa\|Informativa Consenso]] §Ciclo di vita
- [[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|Risposte MF SRS v3]] MF66R65
- [[wiki/analyses/analysis-gap-as-is-to-be\|Analisi Gap AS-IS → TO-BE]] §Stato SCADUTO meccanismo diverso
- Correlato: [[wiki/docs/adr/ADR-015-storicizzazione-immutabile\|ADR-015-storicizzazione-immutabile]] storicizzazione immutabile, [[wiki/docs/adr/ADR-007-batch-01-5min-skip-locked\|ADR-007-batch-01-5min-skip-locked]] BATCH-01 notifica per ANNULLATO
