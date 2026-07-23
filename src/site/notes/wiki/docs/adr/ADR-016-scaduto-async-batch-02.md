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

Workflow BATCH-02 (SC67 risolto — storicizzazione ancorata all'informativa scaduta):
1. Trovare informative scadute (`data_scadenza < NOW()`)
2. Per ogni `cons_id` collegato all'**informativa scaduta A** in stato `ATTIVO`/`NEGATO`:
   - **UPDATE** chiusura record corrente (`data_fine = NOW()`, `login_operazione = 'BATCH_SCADENZA_INF'`)
   - **INSERT** nuovo record storicizzato con stato `SCADUTO` (se `A.annulla_consensi=NO`) o `ANNULLATO` (se `A.annulla_consensi=SI`), `annulla_consensi` letto **dall'informativa scaduta A** e `d_informativa_id = :id_informativa_scaduta` (record ancorato ad A, **non** a una nuova informativa)
3. Se `ANNULLATO` → INSERT in `cons_t_notifica` → BATCH-01 notifica SIA

> ℹ️ La determinazione della "nuova informativa corrente" (SQL canonico MF66R65) **non è necessaria a BATCH-02**: serve alla ri-accettazione del cittadino (CDU-04). Vedi [[wiki/concepts/batch-processes\|Processi Batch]] §ALG02.

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
- SC67 ✅ risolto per via tecnica (storicizzazione ancorata all'informativa scaduta A) — vedi [[wiki/concepts/batch-processes\|Processi Batch]] §ALG02

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

- **SC67 (BAT-02)** — ✅ **risolto per via tecnica (2026-07-23):** il flag `annulla_consensi` si legge dall'informativa **scaduta A** e il record terminale resta ancorato ad A. Allinea l'SQL §7.2 alla prosa autoritativa §6.13; nessun impatto su CDU-17 (snapshot esporta solo consensi attivi). SQL SRS §7.2 corretto (2026-07-23, `.md`+`.docx`, riga variazioni 1.4). Vedi [[wiki/concepts/batch-processes\|Processi Batch]] §ALG02.
- **BAT-03** — 🟠 aperto: CSI considera lo Stato SCADUTO "componente da gestire" → **call dedicata pending** per la gestione asincrona e la comunicazione alle ASR.
- **Frequenza BATCH-02** — ⚪ non vincolante (call 20/07/2026), da concordare in seguito.

## References

- [[wiki/concepts/batch-processes\|Processi Batch — BATCH-01, BATCH-02, BATCH-03]] §BATCH-02
- [[wiki/concepts/ciclo-vita-consenso\|Ciclo di Vita del Consenso]]
- [[wiki/concepts/informativa\|Informativa Consenso]] §Ciclo di vita
- [[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|Risposte MF SRS v3]] MF66R65
- [[wiki/analyses/analysis-gap-as-is-to-be\|Analisi Gap AS-IS → TO-BE]] §Stato SCADUTO meccanismo diverso
- Correlato: [[wiki/docs/adr/ADR-015-storicizzazione-immutabile\|ADR-015-storicizzazione-immutabile]] storicizzazione immutabile, [[wiki/docs/adr/ADR-007-batch-01-5min-skip-locked\|ADR-007-batch-01-5min-skip-locked]] BATCH-01 notifica per ANNULLATO
