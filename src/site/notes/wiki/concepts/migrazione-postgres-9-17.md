---
{"dg-publish":true,"permalink":"/wiki/concepts/migrazione-postgres-9-17/","title":"Migrazione PostgreSQL 9 → 18","tags":["database","postgresql","migrazione","rischio-critico","sprint-0","fase-6"],"dg-note-properties":{"title":"Migrazione PostgreSQL 9 → 18","aliases":["Migrazione PostgreSQL 9 → 17","Migrazione PostgreSQL 9 → 18","Migrazione PG9 PG17","Migrazione PG9 PG18","PG9 → PG17","PG9 → PG18"],"type":"concept","tags":["database","postgresql","migrazione","rischio-critico","sprint-0","fase-6"],"created":"2026-05-15","updated":"2026-08-06","sources":["2026-03-02-appunti-e-pianificazione","2026-03-02-conspref-srs-v1-revised","2026-03-12-pile-tecnologiche-csi","2019-04-08-dizionario-dati-as-is"],"related":["[[conspref-dmp-tracker|CONSPREF-DMP — Tracker Piano Migrazione Dati]]","[[Architettura IaaS]]","[[wiki/sources/2026-03-12-pile-tecnologiche-csi\|Pile Tecnologiche CSI Piemonte]]","[[wiki/analyses/valutazione-qualita-srs-consensi\|Valutazione Qualità SRS — Gestione Consensi]]","[[CSI Piemonte]]"]}}
---


# Migrazione PostgreSQL 9 → 18

**Salto di 9 major release.** Strategia: **dump/restore** (pg_dump custom format → pg_restore). Pianificata in **Fase 6** (Sprint 9) del progetto.

> ⚠️ **Rischio critico:** Il documento formale di piano migrazione ([[wiki/analyses/conspref-dmp-tracker\|CONSPREF-DMP]]) **non è ancora formalizzato**. Senza piano, Fase 6 è a rischio slittamento (vedi [[wiki/analyses/valutazione-qualita-srs-consensi\|Valutazione Qualità SRS — Gestione Consensi]] §Rischio Critico 1). ✅ Responsabile redazione: **[[wiki/entities/csi-piemonte\|CSI Piemonte]]** (confermato 16/07/2026 — GOV-03 chiuso).

> 🔄 **Aggiornamento 06/08/2026 (call CSI):** target di migrazione **aggiornato da PG17 a PG18** — CSI ha confermato di aver nel frattempo migrato lo stack DBaaS a PostgreSQL 18. Strategia e procedura cutover restano invariate.

---

## Contesto stack

| Versione      | Stato CSI                                     | Note               |
| ------------- | --------------------------------------------- | ------------------ |
| PostgreSQL **9.6** (istanza TEST) | **RETIRED** da [[wiki/entities/csi-piemonte\|CSI Piemonte]] | Sorgente dati reale della migrazione |
| PostgreSQL 17 | superato — era CURRENT dic 2025               | ~~DBaaS Nivola (DEV/pre-prod)~~ target originale, sostituito |
| PostgreSQL 18 | **CURRENT** (confermato 06/08/2026)           | DBaaS Nivola (DEV/pre-prod) — nuovo target |

Vedi [[wiki/sources/2026-03-12-pile-tecnologiche-csi\|Pile Tecnologiche CSI Piemonte]] per dettaglio (fotografia 03/2026, PG17 CURRENT all'epoca).

> 🔄 **07/2026 (email CSI):** un **ribaltamento dei dati** dal DB **TEST PostgreSQL 9.6** verrà caricato sul DB **DEV** → baseline con **dati reali** per collaudare la strategia dump/restore prima della Fase 6. (In questa fase esistono solo DEV e pre-prod, non PROD.) DB DEV ora su **PG18** (era PG17, vedi aggiornamento 06/08/2026 sopra).

---

## Strategia cutover (da [[wiki/sources/2026-03-02-appunti-e-pianificazione\|Appunti Sistema + Pianificazione Progetto Consensi]] §Piano Migrazione)

```
1. Snapshot e blocco scritture PG9 PROD
2. Dump finale PG9 PROD (pg_dump -Fc, custom format)
3. Restore su PG18 PROD (pg_restore con jobs parallel)
4. Aggiornare la configurazione applicativa (DSN PG18) via ADA/Chef
5. Restart dell'applicazione (istanze backend)
6. Smoke test funzionale + verifica conteggi tabelle
7. Standby rollback 48h (PG9 mantenuto in read-only)
```

**Finestra cutover:** stimata 21 giorni Fase 6 (Sprint 9). Vincolata da volumi reali (ignoti finché audit DDL non completato).

---

## Rischi tecnici principali

| Area | Rischio | Mitigazione |
|---|---|---|
| **Autenticazione** | md5 → scram-sha-256 (già in PG13+, confermato in PG18) | Aggiornare JDBC driver + datasource pool config |
| **Tipi deprecati** | `money`, `xml`, `OIDs` interni cambi semantica | Audit DDL PG9 per individuare uso |
| **Timestamp** | comportamento `timestamp without time zone` raffinato in versioni intermedie | Test regression su CDU stato |
| **Estensioni** | `gen_random_uuid()` nativo solo da PG13+ (era `pgcrypto` extension) | Cleanup script extension AS-IS |
| **Collation** | ICU default da PG13+ può cambiare ordinamenti | Forzare collation legacy se necessario |
| **Parametri server** | `synchronous_commit`, `wal_level` rifrazionati | Confronto postgresql.conf PG9 vs PG18 |

---

## Audit DDL Sprint 0 (prerequisito)

Vedi `analysis-2026-05-06-checklist-avvio-progetto.md` §A6 e [[wiki/sources/2019-04-08-dizionario-dati-as-is\|Modello Dizionario Dati AS-IS (2019)]] §Note.

Step:
1. Accesso DB PG9 AS-IS (richiesta credenziali a [[wiki/entities/csi-piemonte\|CSI Piemonte]])
2. `\d` su ogni tabella → DDL reale
3. Focus su `cons_s_consenso` (storicizzazione) — verifica struttura fisica vs DDL atteso TO-BE
4. Inventario indici, FK, trigger, sequence
5. Dimensionamento volumi (`pg_total_relation_size`) per stimare durata dump/restore

---

## Modello dati TO-BE post-migrazione

25 tabelle (§8.3 SRS) + 11 [PROPOSTA] (§8.4, incl. cons_t_client_ente) — vs 12 tabelle AS-IS. Vedi [[wiki/sources/2026-03-02-sommario-srs-consensi\|Sommario SRS Gestione Consensi — Indice Strutturale]] per elenco completo.

Trasformazioni schema PG9 → PG18:
- Conversione struttura storicizzazione (UPDATE+INSERT vs trigger storico AS-IS)
- Nuove tabelle: `cons_t_traccia_serv_est` (§8.4.1), `cons_d_asr_destinazione` (§8.4.7), `cons_t_batch_errori` (§8.4.9)
- Estensioni colonne in `cons_s_consenso`, `cons_t_consenso`, `cons_d_informativa`, `cons_t_notifica`, `cons_r_asr_endpoint`

---

## Funzionalità PG17+ sfruttate dal TO-BE (disponibili anche in PG18)

| Funzionalità | Uso |
|---|---|
| `SELECT FOR UPDATE SKIP LOCKED` | [[wiki/concepts/batch-processes\|BATCH-01]] concorrenza-safe multi-pod |
| `gen_random_uuid()` nativa | UUID nuovi record storicizzati (BATCH-02) |
| MERGE statement (SQL standard PG15+) | Upsert in CDU-03 acquisizione |
| `pg_stat_io` | Monitoraggio I/O DBaaS |

---

## Stato e prossimi passi

- ❌ CONSPREF-DMP bozza v1 — **Da produrre Sprint 0** (redazione in carico a [[wiki/entities/csi-piemonte\|CSI Piemonte]] — confermato 16/07/2026)
- ❌ Audit DDL PG9 — Pianificato Sprint 0
- ✅ Responsabile formale: [[wiki/entities/csi-piemonte\|CSI Piemonte]] — confermato 16/07/2026 (GOV-03 **chiuso** in [[wiki/analyses/analysis-2026-05-14-punti-aperti-csi\|Punti Aperti CSI — Tracker Unificato]])
- ✅ Stack target confermato (Q&A CSI #10) — **versione aggiornata a PG18** (call 06/08/2026)
- ✅ Strategia dump/restore accettata

Vedi anche: [[conspref-dmp-tracker|CONSPREF-DMP — Tracker Piano 