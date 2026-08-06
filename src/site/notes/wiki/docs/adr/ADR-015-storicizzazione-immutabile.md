---
{"dg-publish":true,"permalink":"/wiki/docs/adr/adr-015-storicizzazione-immutabile/","title":"Storicizzazione immutabile dei consensi (no sovrascrittura, UPDATE+INSERT)","tags":["storicizzazione","audit","modello-dati","cons-s-consenso","cons-t-consenso","immutabilita"],"dg-note-properties":{"adr":15,"title":"Storicizzazione immutabile dei consensi (no sovrascrittura, UPDATE+INSERT)","status":"accepted","date":"2026-03-02","deciders":["CSI Piemonte","Exprivia"],"supersedes":[],"superseded-by":[],"tags":["storicizzazione","audit","modello-dati","cons-s-consenso","cons-t-consenso","immutabilita"],"related_wiki":["[[wiki/concepts/ciclo-vita-consenso\|Ciclo di Vita del Consenso]]","[[Gestione Consensi - Applicativo]]","[[wiki/analyses/analysis-gap-as-is-to-be\|Analisi Gap AS-IS → TO-BE]]"],"sources":["[[wiki/sources/2026-03-02-conspref-srs-v1-revised\|CONSPREF-SRS-V1.0 revised bozza v2]] §5, §8.3"]}}
---


# ADR-015: Storicizzazione immutabile dei consensi

## Status

`accepted` — vincolo TO-BE formalizzato in SRS §5 (macchina a stati) e §8.3 (modello dati).

## Context

Un consenso sanitario è un atto formale con valore legale (GDPR, DPCM 178/2015, Nota Garante 0020885/2017). Requisito:

- Ogni atto di consenso deve essere **tracciabile nel tempo**
- Una variazione successiva (rilascio → modifica → revoca → riaccettazione) **non deve cancellare** lo stato precedente
- L'audit legale deve poter ricostruire la timeline completa di un cittadino

AS-IS: tabella `cons_s_consenso` (storico) esiste ma **non è popolata**. La sovrascrittura sull'attivo non lascia traccia. Rischio di compliance.

## Decision

Vincolo: **nessuna sovrascrittura dei record di consenso**. Ogni variazione produce:

1. **UPDATE** sul record corrente: `data_fine = NOW()`, `data_modifica = NOW()`, `login_operazione = <attore>`
2. **INSERT in `cons_s_consenso`** copia del record pre-variazione (storicizzato)
3. **INSERT in `cons_t_consenso`** nuovo record con il nuovo stato

Vale per **tutte** le transizioni:
- ATTIVO ↔ NEGATO (cambio diretto via CDU-04/05/10/11)
- ATTIVO/NEGATO → SCADUTO/ANNULLATO (BATCH-02, vedi [[wiki/docs/adr/ADR-016-scaduto-async-batch-02\|ADR-016-scaduto-async-batch-02]])
- SCADUTO/ANNULLATO → ATTIVO/NEGATO (riaccettazione informativa)

Per UUID dei nuovi record storicizzati: `gen_random_uuid()` nativa PostgreSQL 18 (disponibile da PG13+, [[wiki/docs/adr/ADR-001-stack-tecnologico\|ADR-001-stack-tecnologico]]).

## Consequences

### Positive
- Audit legale completo: tutta la storia di un consenso ricostruibile
- Compliance GDPR / DPCM 178/2015 nativa
- Reporting (es. quanti cittadini hanno mai negato un consenso) banale
- Recovery / investigation post-bug: si vede esattamente cosa è successo

### Negative
- Crescita lineare `cons_s_consenso` (mai purge) → policy retention da definire (vedi [[wiki/concepts/informativa\|Informativa Consenso]] §Punti aperti)
- Query stato corrente richiede `WHERE data_fine IS NULL` o ultimo record per `(cf, tipo, sotto_tipo, ente)`
- Audit DDL Sprint 0 deve verificare struttura `cons_s_consenso` AS-IS (potrebbe non corrispondere a DDL TO-BE attesa)

### Neutral
- Trigger DB AS-IS (se presenti) da rimuovere — storicizzazione applicativa, non DB-side

## Alternatives considered

| Alternativa | Motivo scarto |
|---|---|
| UPDATE in-place (AS-IS) | Niente audit; compliance GDPR a rischio |
| Trigger storicizzazione DB-side | Logica nascosta lato DB, difficile da testare/versionare; preferenza per logica applicativa esplicita |
| Event sourcing puro | Eccessivo: tabella `cons_t_consenso` + `cons_s_consenso` copre il fabbisogno senza nuovo paradigma |

## Open issues

- Policy retention di `cons_s_consenso` non definita (vedi [[wiki/concepts/informativa\|Informativa Consenso]] §Punti aperti)
- Audit DDL `cons_s_consenso` AS-IS bloccato (TECH-01)

## References

- [[wiki/concepts/ciclo-vita-consenso\|Ciclo di Vita del Consenso]] §Regola fondamentale — No sovrascrittura
- [[wiki/sources/2026-03-02-conspref-srs-v1-revised\|CONSPREF-SRS-V1.0 revised bozza v2]] §5 (macchina a stati), §8.3 (modello dati)
- [[wiki/analyses/analysis-gap-as-is-to-be\|Analisi Gap AS-IS → TO-BE]] §Storicizzazione — attivazione feature
- Correlato: [[wiki/docs/adr/ADR-016-scaduto-async-batch-02\|ADR-016-scaduto-async-batch-02]] SCADUTO via BATCH-02 (anch'esso storicizzato), [[wiki/docs/adr/ADR-001-stack-tecnologico\|ADR-001-stack-tecnologico]] stack (PG18 `gen_random_uuid` nativa)
