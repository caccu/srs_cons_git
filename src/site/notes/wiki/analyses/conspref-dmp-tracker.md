---
{"dg-publish":true,"permalink":"/wiki/analyses/conspref-dmp-tracker/","title":"CONSPREF-DMP — Tracker Piano Migrazione Dati","tags":["tracker","dmp","migrazione","rischio-critico","governance","sprint-0"],"dg-note-properties":{"title":"CONSPREF-DMP — Tracker Piano Migrazione Dati","aliases":["CONSPREF-DMP — Tracker Piano Migrazione Dati","CONSPREF-DMP Tracker","CONSPREF-DMP"],"type":"analysis","tags":["tracker","dmp","migrazione","rischio-critico","governance","sprint-0"],"created":"2026-05-15","updated":"2026-07-16","sources":["2026-03-02-conspref-srs-v1-revised","2026-03-02-appunti-e-pianificazione","2026-03-02-domande-srs-csi-v02"],"related":["[[wiki/concepts/migrazione-postgres-9-17\|Migrazione PostgreSQL 9 → 17]]","[[wiki/analyses/valutazione-qualita-srs-consensi\|Valutazione Qualità SRS — Gestione Consensi]]","[[wiki/entities/exprivia\|Exprivia S.p.A.]]","[[CSI Piemonte]]","[[wiki/analyses/analysis-2026-05-06-checklist-avvio-progetto\|Checklist Avvio Progetto]]","[[wiki/analyses/analysis-2026-05-14-punti-aperti-csi\|Punti Aperti CSI — Tracker Unificato]]"]}}
---


# CONSPREF-DMP — Tracker Piano Migrazione Dati

**Stato:** 🔴 **Non formalizzato**. Rischio critico per Fase 6 (Sprint 9) migrazione [[wiki/concepts/migrazione-postgres-9-17\|PG9 → PG18]] (target aggiornato da PG17, call CSI 06/08/2026). ✅ **Responsabile redazione: [[wiki/entities/csi-piemonte\|CSI Piemonte]]** (confermato 16/07/2026 — GOV-03 chiuso).

> Questo tracker centralizza lo stato del documento CONSPREF-DMP e i decision-point aperti. Non sostituisce il documento stesso (redazione in carico a [[wiki/entities/csi-piemonte\|CSI Piemonte]], con input tecnici [[wiki/entities/exprivia\|Exprivia S.p.A.]], in Sprint 0).

---

## Cos'è CONSPREF-DMP

**Data Migration Plan** — documento formale di pianificazione migrazione dati AS-IS → TO-BE che deve coprire:

| Sezione | Contenuto |
|---|---|
| Inventario sorgente | DDL reale PG9 + volumi + indici + FK |
| Mapping AS-IS → TO-BE | Tabella-per-tabella, campo-per-campo (12 → 25 tabelle) |
| Trasformazioni | UPDATE+INSERT vs trigger storico, conversioni tipo, nuovi UUID |
| Strategia cutover | dump/restore, finestra, rollback |
| Test plan | Verifica conteggi, integrità referenziale, smoke test funzionale |
| Rollback | Standby PG9 read-only 48h post-cutover |
| Responsabilità | RACI Exprivia ↔ CSI ↔ Regione |

---

## Stato di ciascuna sezione

| #   | Sezione               | Stato                                                                                     | Owner                          | Sprint target |
| --- | --------------------- | ----------------------------------------------------------------------------------------- | ------------------------------ | ------------- |
| 1   | Inventario sorgente   | ❌ Bloccato — manca accesso DB PG9                                                         | [[wiki/entities/csi-piemonte\|csi-piemonte]]               | Sprint 0      |
| 2   | Mapping AS-IS → TO-BE | 🟡 Parziale — §8 SRS copre TO-BE, AS-IS scarno (vedi [[wiki/sources/2019-04-08-dizionario-dati-as-is\|2019-04-08-dizionario-dati-as-is]] | [[wiki/entities/csi-piemonte\|CSI Piemonte]] | Sprint 0      |
| 3   | Trasformazioni        | ❌ Da produrre                                                                             | [[wiki/entities/csi-piemonte\|CSI Piemonte]] | Sprint 1      |
| 4   | Strategia cutover     | 🟡 Bozza nei [[wiki/sources/2026-03-02-appunti-e-pianificazione\|2026-03-02-appunti-e-pianificazione]]                                      | [[wiki/entities/csi-piemonte\|csi-piemonte]]               | Sprint 0      |
| 5   | Test plan             | ❌ Da produrre                                                                             | [[wiki/entities/exprivia\|Exprivia S.p.A.]]  | Sprint 8      |
| 6   | Rollback              | 🟡 Bozza (standby 48h)                                                                    | [[wiki/entities/exprivia\|Exprivia S.p.A.]]  | Sprint 8      |
| 7   | RACI                  | 🟡 Responsabile redazione confermato: CSI Piemonte (16/07/2026); RACI di dettaglio da produrre | [[wiki/entities/csi-piemonte\|CSI Piemonte]] | Sprint 0      |

---

## Decision-point aperti

### ~~D1 — Responsabile formale lato CSI~~ ✅ CHIUSO 16/07/2026
Redazione CONSPREF-DMP **in carico a [[wiki/entities/csi-piemonte\|CSI Piemonte]]**.
- **Domanda 11** in [[wiki/sources/2026-03-02-domande-srs-csi-v02\|Domande SRS Consensi — Revisione CSI V02]]: risposta storica = "Non ancora formalizzato"
- Tracciato come **GOV-03** (chiuso) in [[wiki/analyses/analysis-2026-05-14-punti-aperti-csi\|Punti Aperti CSI — Tracker Unificato]]

### D2 — Strategia logical replication vs dump/restore puro
- **Default attuale:** dump/restore (pg_dump -Fc + pg_restore)
- **Alternativa:** logical replication (CDC) per ridurre downtime — non valutata
- **Da decidere:** vincoli SLA downtime applicativo

### D3 — Conversione tipi deprecati
- `money`, `xml` se presenti in PG9 → tipo target?
- Dipende da audit DDL Sprint 0

### D4 — Retention storico
- Informative scadute (vedi [[wiki/concepts/informativa\|Informativa Consenso]]) — policy retention non definita per audit legale

---

## Rischi correlati

| Codice | Rischio | Riferimento |
|---|---|---|
| RISCH-01 | DMP non formalizzato → slittamento Fase 6 | [[wiki/analyses/valutazione-qualita-srs-consensi\|Valutazione Qualità SRS — Gestione Consensi]] §RC1 |
| GOV-03 | ~~Responsabile CSI mancante~~ ✅ chiuso 16/07/2026 (CSI Piemonte) | [[wiki/analyses/analysis-2026-05-14-punti-aperti-csi\|Punti Aperti CSI — Tracker Unificato]] |
| TECH-01 | Audit DDL PG9 bloccato da accessi | Checklist §A6 |
| TECH-02 | Volumi reali ignoti → finestra cutover stimabile | Da audit |

---

## Azioni immediate (Sprint 0)

1. ~~Assegnare responsabile redazione CONSPREF-DMP~~ ✅ **CSI Piemonte** (confermato 16/07/2026)
2. **Richiedere accesso DB PG9 AS-IS** a [[wiki/entities/csi-piemonte\|CSI Piemonte]] (o lavorare sul ribaltamento dati su DEV, 07/2026)
3. **Avviare audit DDL PG9** (`\d` per ogni tabella, focus `cons_s_consenso`)
4. ~~Identificare sponsor CSI per migrazione~~ ✅ chiuso con GOV-03
5. **Produrre bozza v1 CONSPREF-DMP** entro fine Sprint 0 (CSI, con input tecnici Exprivia)

Vedi [[wiki/analyses/analysis-2026-05-06-checklist-avvio-progetto\|Checklist Avvio Progetto]] §A6 per la versione checklist operativa.

---

## Riferimenti tra wiki

- [[wiki/concepts/migrazione-postgres-9-17\|Migrazione PostgreSQL 9 → 17]] — concept tecnico migrazione
- [[wiki/analyses/valutazione-qualita-srs-consensi\|Valutazione Qualità SRS — Gestione Consensi]] — RC1 sul DMP mancante
- [[wiki/entities/exprivia\|Exprivia S.p.A.]] — fornitore, input tecnici al DMP (redazione in carico a [[wiki/entities/csi-piemonte\|CSI Piemonte]])

---

## ADR correlati

| ADR | Decisione |
|---|---|
| [ADR-013](ADR-013-migrazione-pg-dump-restore.md) | Strategia migrazione dump/restore (questo tracker monitora la formalizzazione del DMP) |
| [ADR-003](ADR-003-dbaas-nivola.md) | DBaaS Nivola (vincolo provisioning + no-filesystem) |
