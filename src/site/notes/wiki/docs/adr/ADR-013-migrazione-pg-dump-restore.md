---
{"dg-publish":true,"permalink":"/wiki/docs/adr/adr-013-migrazione-pg-dump-restore/","title":"Migrazione PostgreSQL 9 → 18 via dump/restore logico","tags":["database","postgresql","migrazione","dump-restore","sprint-9","rischio-critico"],"dg-note-properties":{"adr":13,"title":"Migrazione PostgreSQL 9 → 18 via dump/restore logico","status":"accepted","date":"2026-03-02","updated":"2026-08-06","deciders":["Exprivia","CSI Piemonte"],"supersedes":[],"superseded-by":[],"tags":["database","postgresql","migrazione","dump-restore","sprint-9","rischio-critico"],"related_wiki":["[[wiki/concepts/migrazione-postgres-9-17\|Migrazione PostgreSQL 9 → 17]]","[[wiki/analyses/conspref-dmp-tracker\|CONSPREF-DMP — Tracker Piano Migrazione Dati]]","[[valutazione-qualita-srs-consensi|Valutazione Qualità SRS — Gestione Consensi]]"],"sources":["[[wiki/sources/2026-03-02-appunti-e-pianificazione\|Appunti e pianificazione]] §Piano Migrazione","[[wiki/analyses/analysis-2026-05-06-checklist-avvio-progetto\|Checklist Avvio Progetto]] §A6"]}}
---


# ADR-013: Strategia migrazione PG9 → PG18 — dump/restore

## Status

`accepted` — strategia confermata in appunti e checklist; CONSPREF-DMP formale **non ancora prodotto** (rischio aperto); redazione in carico a **CSI Piemonte** (confermato 16/07/2026 — GOV-03 chiuso).

> 🔄 **Aggiornamento 06/08/2026 (call CSI):** CSI ha confermato di aver **migrato lo stack DBaaS a PostgreSQL 18** (nel frattempo divenuto CURRENT, superando PG17). Il target della migrazione dati del progetto CONSPREF **passa da PG17 a PG18**. Strategia dump/restore e procedura cutover **invariate**; da rivalutare in Sprint 0 eventuali novità PG17→PG18 nell'audit DDL (autenticazione, tipi deprecati, estensioni — già in ambito dell'analisi salto multi-release).

## Context

PostgreSQL 9 è RETIRED da CSI; PG18 è CURRENT (DBaaS Nivola, confermato 06/08/2026 — in precedenza PG17). Salto di **9 major release**, con cambi semantici e di sicurezza significativi:

- Autenticazione: `md5` → `scram-sha-256`
- Tipi deprecati: `money`, `xml`, `OIDs` interni
- Estensioni: `gen_random_uuid()` nativa da PG13+ (era `pgcrypto`)
- Collation: ICU default da PG13+
- Comportamento `timestamp without time zone` raffinato

Vincoli:
- DBaaS Nivola ([[wiki/docs/adr/ADR-003-dbaas-nivola\|ADR-003-dbaas-nivola]]): niente accesso filesystem diretto, niente `pg_upgrade` server-side
- Nivola gestisce backup/HA — la migrazione deve passare per dump/restore client-side
- Finestra cutover stretta da minimizzare per impatto utenti

## Decision

Strategia: **dump/restore logico** (`pg_dump` custom format → `pg_restore`).

Procedura cutover PROD:
1. Snapshot e blocco scritture PG9 PROD
2. `pg_dump -Fc` finale (custom format)
3. `pg_restore` su PG18 PROD con `--jobs N` (parallelism)
4. Aggiornare K8s Secret con DSN PG18
5. Restart pod applicativi (rolling)
6. Smoke test funzionale + verifica conteggi tabelle
7. Standby rollback 48h (PG9 mantenuto in read-only)

Finestra cutover stimata: 21 giorni in Fase 6 (Sprint 9). Vincolata da volumi reali ignoti finché audit DDL non completato.

## Consequences

### Positive
- Funziona senza accesso filesystem (compatibile DBaaS Nivola)
- Trasformazioni schema PG9 → PG18 applicabili in restore (custom format consente filtri)
- Rollback semplice: PG9 read-only standby per 48h post-cutover
- Compatibilità garantita: pg_dump/pg_restore stabili cross-version

### Negative
- **Downtime applicativo durante dump finale + restore** (no logical replication)
- Volumi reali ignoti → durata finestra stimabile solo dopo audit DDL Sprint 0
- **CONSPREF-DMP** formale non prodotto → rischio slittamento Fase 6 (R6 in checklist)
- Niente parallelismo migrazione/applicativo (downtime sincrono)

### Neutral
- Audit DDL PG9 Sprint 0 (richiede accesso DB AS-IS — bloccato in attesa CSI)
- Trasformazioni schema necessarie: storicizzazione, nuove tabelle ([[wiki/docs/adr/ADR-015-storicizzazione-immutabile\|ADR-015-storicizzazione-immutabile]]), estensioni colonne

## Alternatives considered

| Alternativa | Motivo scarto |
|---|---|
| `pg_upgrade` server-side | Incompatibile DBaaS Nivola (no filesystem access); 8 major release jump non sempre supportato |
| Logical replication (CDC) | Non valutata in dettaglio; complica setup; downtime minore ma più rischio di drift dati durante sync |
| Esportazione CSV custom | Rischio perdita di vincoli FK, sequence, indici; restore manuale poco riproducibile |

## Open issues

- ~~GOV-03: responsabile formale lato CSI per migrazione~~ ✅ chiuso 16/07/2026 — CSI Piemonte (tracker [[wiki/analyses/analysis-2026-05-14-punti-aperti-csi\|Punti Aperti CSI]] §8)
- D2 [[wiki/analyses/conspref-dmp-tracker\|CONSPREF-DMP]]: valutare logical replication come alternativa per ridurre downtime
- D3: tipi deprecati (`money`, `xml`) — conversione target dipende da audit DDL Sprint 0
- TECH-01: audit DDL PG9 bloccato da accessi DB

## References

- [[wiki/concepts/migrazione-postgres-9-17\|Migrazione PostgreSQL 9 → 17]] (concept tecnico completo con rischi e features sfruttate)
- [[wiki/analyses/conspref-dmp-tracker\|CONSPREF-DMP — Tracker Piano Migrazione Dati]] (governance e decision-point aperti)
- [[wiki/sources/2026-03-02-appunti-e-pianificazione\|Appunti e pianificazione]] §Piano Migrazione
- [[wiki/analyses/analysis-2026-05-06-checklist-avvio-progetto\|Checklist Avvio Progetto]] §A6
- Correlato: [[wiki/docs/adr/ADR-001-stack-tecnologico\|ADR-001-stack-tecnologico]] stack (PG18), [[wiki/docs/adr/ADR-003-dbaas-nivola\|ADR-003-dbaas-nivola]] DBaaS Nivola (vincolo accesso)
