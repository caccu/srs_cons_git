---
{"dg-publish":true,"permalink":"/wiki/docs/adr/readme/","title":"Architecture Decision Records — Gestione Consensi","dg-note-properties":{"title":"Architecture Decision Records — Gestione Consensi","type":"index","created":"2026-05-19","updated":"2026-05-19"}}
---


# Architecture Decision Records — Indice

Registry delle decisioni architetturali del progetto **Gestione Consensi Regione Piemonte**. Ogni ADR cattura una decisione che chiude opzioni concrete, con conseguenze tecniche durature. Formato: Nygard-style con frontmatter YAML per machine-readability.

**Convenzioni:**
- Numerazione sequenziale `ADR-NNN` (zero-padded a 3 cifre)
- Status: `proposed` → `accepted` → (`deprecated` | `superseded by [ADR-MMM]`)
- ADR non si modificano post `accepted` — si aggiunge un nuovo ADR che supersede
- Cross-link wiki: ogni ADR linka concept/analysis pages in `wiki/` rilevanti; viceversa le pages elencano gli ADR pertinenti

---

## Registry

| # | Titolo | Status | Data | Tema |
|---|---|---|---|---|
| [ADR-001](ADR-001-stack-tecnologico.md) | Stack tecnologico (Spring Boot 3 + Java 17 + Angular 19 + PG17) | accepted | 2026-03-02 | Stack |
| [ADR-002](ADR-002-piattaforma-ecaas.md) | Piattaforma ECaaS Kubernetes Nivola + vincoli | superseded | 2026-03-02 | Infrastruttura |
| [ADR-003](ADR-003-dbaas-nivola.md) | DBaaS Nivola esterno al namespace | accepted | 2026-03-02 | Infrastruttura |
| [ADR-004](ADR-004-no-api-gateway.md) | No API Gateway — sicurezza applicativa via Spring Security | partially-superseded | 2026-03-02 | Sicurezza |
| [ADR-005](ADR-005-sicurezza-cdu-15-16.md) | Sicurezza CDU-15/16 — OAuth2 CC + JWT + autorizzazione per ente | accepted | 2026-05-14 | Sicurezza |
| [ADR-006](ADR-006-batch-03-pull-cdu-17.md) | Sostituzione BATCH-03 push → CDU-17 PULL (centro stella) | **proposed** | 2026-05-14 | Batch / Integrazione |
| [ADR-007](ADR-007-batch-01-5min-skip-locked.md) | BATCH-01 5 min con SKIP LOCKED | accepted | 2026-05-14 | Batch |
| [ADR-008](ADR-008-ssot-form-renderer.md) | SSoT Form Renderer unico (Cittadino + Operatore) | accepted | 2026-05-14 | UX / Architettura |
| [ADR-009](ADR-009-eliminazione-sistemats.md) | Eliminazione SistemaTS dall'integrazione | accepted | 2026-05-14 | Integrazione |
| [ADR-010](ADR-010-cdu-01-split.md) | Split CDU-01 in CDU-01a Operatore + CDU-01b Cittadino | accepted | 2026-05-14 | CDU |
| [ADR-011](ADR-011-merge-cdu-04-05-cittadino.md) | Merge CDU-04/CDU-05 lato Cittadino — pulsante unico | accepted | 2026-05-14 | UX / CDU |
| [ADR-012](ADR-012-notificatore-deleghe-post-completato.md) | Notifica via Notificatore di Deleghe (non UNP), post-COMPLETATO | accepted | 2026-05-14 | Notifica |
| [ADR-013](ADR-013-migrazione-pg-dump-restore.md) | Migrazione PG9 → PG17 via dump/restore | accepted | 2026-03-02 | Database / Migrazione |
| [ADR-014](ADR-014-apache-cxf-soap-client.md) | Apache CXF come client SOAP | accepted | 2026-03-02 | Integrazione |
| [ADR-015](ADR-015-storicizzazione-immutabile.md) | Storicizzazione immutabile (no sovrascrittura) | accepted | 2026-03-02 | Modello dati |
| [ADR-016](ADR-016-scaduto-async-batch-02.md) | Stato SCADUTO async via BATCH-02 | accepted | 2026-03-02 | Batch / Semantica |
| [ADR-017](ADR-017-lis-terzo-canale.md) | LIS terzo canale acquisizione consenso | accepted | 2026-05-14 | Canali |
| [ADR-018](ADR-018-rfc-7807-error-response.md) | RFC 7807 error response | accepted | 2026-03-02 | API |
| [ADR-019](ADR-019-cdu-06-pdf-scope-ridotto.md) | CDU-06 PDF — scope ridotto, no firma | accepted | 2026-05-14 | CDU |

---

## Stato decisioni per categoria

| Categoria | Accepted | Proposed | Deprecated/Superseded |
|---|---|---|---|
| Stack | 1 | 0 | 0 |
| Infrastruttura | 1 | 0 | 1 |
| Sicurezza | 1 | 0 | 1 |
| Batch | 2 | 1 | 0 |
| Integrazione | 3 | 0 | 0 |
| Modello dati | 2 | 0 | 0 |
| UX / CDU | 4 | 0 | 0 |
| API | 1 | 0 | 0 |
| Notifica | 1 | 0 | 0 |
| Canali | 1 | 0 | 0 |
| **Totale** | **16** | **1** | **2** |

L'unico `proposed` (ADR-006 CDU-17 PULL) attende formale sign-off CSI Piemonte; decisione propagata internamente sulla wiki ma non ancora confermata cliente.

---

## Decision-graph (dipendenze concettuali)

```
ADR-001 stack
  ├── ADR-002 ECaaS
  │     └── ADR-003 DBaaS Nivola
  │           └── ADR-013 migrazione PG9 → PG17
  ├── ADR-014 Apache CXF
  └── ADR-007 BATCH-01 5min (SKIP LOCKED nativo PG17)

ADR-004 no API GW
  └── ADR-005 sicurezza CDU-15/16
        └── ADR-006 CDU-17 PULL (riusa pattern)

ADR-008 SSoT Form Renderer
  ├── ADR-011 merge CDU-04/05 cittadino (pulsante unico)
  └── ADR-017 LIS (riusa renderer)

ADR-015 storicizzazione immutabile
  └── ADR-016 SCADUTO async via BATCH-02 (storicizzato)

ADR-018 RFC 7807
  └── ADR-005 + ADR-006 (errori applicativi)

ADR-010 split CDU-01
  └── ADR-009 no SistemaTS (CDU-07 a valle CDU-01a)

ADR-012 Notificatore Deleghe post-COMPLETATO
  └── ADR-007 BATCH-01 (precondizione COMPLETATO)

ADR-019 CDU-06 PDF scope ridotto (standalone, allineato con UX cittadino)
```

---

## Punti aperti correlati

Vedi tracker autoritativo: [[wiki/analyses/analysis-2026-05-14-punti-aperti-csi\|analysis-2026-05-14-punti-aperti-csi]].

ADR con open issues:
- **ADR-005**: SEC-01÷06 (URL AS, JWT, onboarding SIA, TTL, scope, revoca)
- **ADR-006**: PULL-01÷07 (variante race, canale notifica, scope, page_size, conferma, deprecation)
- **ADR-007**: BAT-01 (WSDL SRV-01 vs SRV-03)
- **ADR-010**: ID-01 (GASP Salute protocollo)
- **ADR-013**: ~~GOV-03 (responsabile CSI migrazione)~~ ✅ chiuso 16/07/2026 (CSI Piemonte), TECH-01 (audit DDL PG9)
- **ADR-016**: SC67 (logica INSERT storicizzazione BATCH-02), BAT-03 (comunicazione ASR cambio semantica)
- **ADR-017**: INT-03 (acronimo LIS + spec integrazione)
- **ADR-019**: GOV-02 (validazione `[PROPOSTA]` struttura PDF)

---

## Workflow ADR

1. **Nuova decisione architetturale** → bozza ADR con `status: proposed`
2. **Discussione + validazione** → status → `accepted` + data + deciders
3. **Decisione modificata** → nuovo ADR con `supersedes: [ADR-NNN]`; il vecchio diventa `superseded-by: [ADR-MMM]`
4. **Decisione abbandonata** → `deprecated` con motivazione
5. **Verifica coerenza** → `adr verify` (ruflo-adr skill) per dangling refs / supersede cycles
