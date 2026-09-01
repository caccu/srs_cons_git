---
{"dg-publish":true,"permalink":"/wiki/index/","title":"Wiki Index","tags":["gardenEntry"],"dg-note-properties":{"title":"Wiki Index","aliases":["Wiki Index"],"type":"index","updated":"2026-08-06"}}
---


# Wiki Index

Master catalogo di tutte le pagine. Aggiornato.

> 🔴 **Perimetro progetto (call CSI 06/08/2026, [[wiki/docs/adr/ADR-021-perimetro-solo-operatore\|ADR-021]]):** questo progetto sviluppa **solo la Webapp Operatore**. La Webapp Cittadino esiste ma è fuori dal perimetro di sviluppo. Pagine/CDU marcati ❌ OUT sotto restano come contesto storico.

---

## Rischi Critici Aperti (quick reference)

| Rischio                                                                             | Impatto                                              | Azione                                                                                        |
| ----------------------------------------------------------------------------------- | ---------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| ~~GASP Salute: OIDC vs SAML2~~ — ✅ **CHIUSO** SAML2 confermato (verbale 11/06/2026) | CDU-01 può procedere alla progettazione              | —                                                                                             |
| [[wiki/analyses/conspref-dmp-tracker\|conspref-dmp-tracker]] non formalizzato                                           | Slittamento Fase 6 migrazione PG9→PG18 (target aggiornato da PG17, 06/08/2026) | ✅ Responsabile: **CSI Piemonte** (16/07/2026 — GOV-03 chiuso); sollecitare bozza v1 Sprint 0 |
| OpenAPI CDU-15/16 — [[wiki/analyses/analysis-2026-05-06-openapi-cdu-15-16\|analysis-2026-05-06-openapi-cdu-15-16]], 5 TBD aperti con CSI | Blocca go-live integrazione SIA ASR                  | Condividere bozza con ASR, confermare 5 TBD Sprint 1-2                                        |
| BATCH-01: SRV-01 vs SRV-03?                                                         | Errore implementativo WSDL                           | Conferma scritta da CSI                                                                       |
| SCADUTO AS-IS ≠ TO-BE (BAT-03) — [[wiki/analyses/analysis-2026-08-05-stato-scaduto-comunicazione-sia\|analysis-2026-08-05-stato-scaduto-comunicazione-sia]] | Rottura integrazione SIA ASR (omonimia: AS-IS = record superato, TO-BE = record corrente) | Call CSI dedicata pending: scegliere canale async (racc. delta PULL CDU-17); documentare diff in OpenAPI |
| ~~BATCH-03 push contestato TR34~~ ✅ risolto                                        | Onere infra regionale, cliente vuole "centro stella" | PULL CDU-17 ([[wiki/docs/adr/ADR-006-batch-03-pull-cdu-17\|ADR-006-batch-03-pull-cdu-17]], **accepted**) — **confermato dal committente 20/07/2026**, Variante B eliminata |
| SC67 — INSERT storicizzazione BATCH-02                                              | "Da approfondire e verificare meglio"                | Discutere con CSI prima di chiudere SRS                                                       |

Dettaglio completo: [[wiki/analyses/valutazione-qualita-srs-consensi\|Valutazione Qualità SRS — Gestione Consensi]]

---

## Entities

- [[wiki/entities/csi-piemonte\|CSI Piemonte]] — Committente tecnico, gestore infrastruttura IaaS/Nivola, fornitore componenti (GASP, AURA, PUA, IRIS, UNP)
- [[wiki/entities/exprivia\|Exprivia S.p.A.]] — Fornitore sviluppo, redattore SRS, team analisi e sviluppo TO-BE
- [[wiki/entities/notificatore-unp\|Notificatore UNP]] — User Notification Platform CSI (REST); distinto da Notificatore di Deleghe; canali email/push/IO/mex
- [[wiki/entities/regione-piemonte\|Regione Piemonte]] — Committente finale, proprietaria Sanità Elettronica Regionale

---

## Concepts

- [[wiki/concepts/gestione-consensi-applicativo\|Gestione Consensi - Applicativo]] — Sistema centrale consensi sanitari Regione Piemonte: 3 livelli, 3 profili (Cittadino OUT, **Operatore unico** — non più 2, corretto 06/08/2026 —, SIA), 17 CDU, 2 batch (BATCH-03 rimosso → CDU-17 PULL); 2 canali di acquisizione (LIS non è un terzo canale — ADR-020, 06/08/2026); **solo Webapp Operatore in scope di sviluppo — ADR-021, 06/08/2026**
- [[wiki/concepts/ciclo-vita-consenso\|Ciclo di Vita del Consenso]] — Macchina a stati: NON_ESPRESSO → ATTIVO/NEGATO → SCADUTO/ANNULLATO; no sovrascrittura
- [[wiki/concepts/architettura-iaas\|Architettura IaaS]] — Infrastruttura IaaS Nivola CSI (non ECaaS/Kubernetes), provisioning CSI; deploy ADA/Chef; 07/2026: ambienti DEV + pre-prod (PROD in fase successiva)
- [[wiki/concepts/gasp-salute\|GASP Salute]] — Identity Provider federato SPID/CIE; protocollo **SAML2** confermato (verbale 11/06/2026); ❌ **fuori scope di sviluppo dal 06/08/2026** (serviva solo Webapp Cittadino — ADR-021)
- [[wiki/concepts/batch-processes\|Processi Batch — BATCH-01, BATCH-02, BATCH-03]] — Notifica/scadenza/allineamento asincrono; ⚠️ BATCH-01 ambiguità WSDL SRV-01 vs SRV-03
- [[wiki/concepts/sistemi-esterni-integrati\|Sistemi Esterni Integrati]] — AURA, SIA ASR, Notificatore UNP, Gestione Deleghe, PUA/Configuratore, LIS/RIS (integrazione BE esistente, non canale — ADR-020); stato approvvigionamento Sprint 0
- [[wiki/concepts/sicurezza-cdu-15-16\|Sicurezza CDU-15-16 — Modello Autorizzazione per Ente]] — Risposta TR30: OAuth2 Client Credentials + JWT + tabella client_ente + filter Spring Security; AS-IS **no API Manager**, difesa a 3 livelli; TO-BE API Manager CSI per nuovi fruitori
- [[wiki/concepts/alternativa-batch-03-pull\|Alternativa BATCH-03 — PULL CDU-17 (centro stella)]] — Sostituzione BATCH-03 push con PULL REST paginato cursor-based; **hub-and-spoke**, zero push, blocco obbligatorio, riusa security CDU-15/16. **Rielaborato e confermato 20/07/2026** (Variante B eliminata, passo 5, endpoint CRUD via APIM, scenario manutenzione)
- [[wiki/concepts/composizione-dinamica-form-consenso\|Composizione Dinamica Form Consenso — Single Source of Truth]] — Pattern SSoT Form Renderer, nato per Citt+Op (CDU-02/03/04/09/10/11); MF26/28/30/35/37/39/41/43/45/57 — **in questo progetto si applica solo a Operatore (CDU-09/10/11) — ADR-021, 06/08/2026**
- [[wiki/concepts/informativa\|Informativa Consenso]] — Oggetto dichiarativo del consenso: modello `cons_d_informativa`, ciclo di vita, lookup CDU-03, base Form Renderer
- [[wiki/concepts/migrazione-postgres-9-17\|Migrazione PostgreSQL 9 → 18]] — Strategia dump/restore Fase 6 Sprint 9; target aggiornato da PG17 a PG18 (06/08/2026); rischi tipi/auth scram/timestamp; audit DDL prerequisito Sprint 0
- [[wiki/concepts/stack-tecnologico-applicativo\|Stack Tecnologico Applicativo]] — Angular 19 + Spring Boot 3.4.10+ / Java 17 + PostgreSQL 18 su IaaS Nivola; vincoli docker-base CSI

---

## Sources

### Specifica TO-BE

- [[wiki/sources/2026-03-02-conspref-srs-v1-revised\|CONSPREF-SRS-V1.0 revised bozza v2]] — SRS TO-BE completo: 17 CDU, architettura, NFR, batch, modello dati 26 tabelle (25 §8.3 + cons_t_client_ente §8.4.11)
- [[wiki/sources/2026-03-02-sommario-srs-consensi\|Sommario SRS Gestione Consensi — Indice Strutturale]] — TOC/mappa strutturale SRS: 26 tabelle TO-BE, 11 proposte §8.4
- [[wiki/sources/2026-03-02-appunti-e-pianificazione\|Appunti Sistema + Pianificazione Progetto Consensi]] — Sintesi architetturale + 8 fasi + 10 sprint + procedura migrazione PG9→PG17 (target storico, aggiornato a PG18 il 06/08/2026 — vedi [[wiki/concepts/migrazione-postgres-9-17\|migrazione-postgres-9-17]])
- [[wiki/sources/2026-03-02-domande-srs-csi-v02\|Domande SRS Consensi — Revisione CSI V02]] — 13 Q&A Exprivia/CSI: no API Gateway, GASP Salute IdP, SOAP AS-IS, DBaaS Nivola

### Requisiti e standard CSI

- [[wiki/sources/2023-09-01-conspref-srs-01-v03\|CONSPREF-SRS-01-V03 Requisiti Gestione Consensi]] — Documento requisiti committente CSI (2023): regole business, backoffice, profili PUA
- [[wiki/sources/2026-03-12-pile-tecnologiche-csi\|Pile Tecnologiche CSI Piemonte]] — Export stack tecnologici approvati CSI: PG17 CURRENT (fotografia 03/2026), PG9 RETIRED — PG17 superato da PG18 il 06/08/2026, vedi [[wiki/concepts/migrazione-postgres-9-17\|migrazione-postgres-9-17]]
- [[wiki/sources/2019-06-01-linea-guida-fornitori-cloud-native\|Linee Guida Cloud Native per Fornitori v1.0.1]] — Vincoli ECaaS vincolanti: TRAEFIK, Artifactory, Dockerfile rules, CI/CD, health probes

### Diagrammi e artefatti tecnici

- [[wiki/sources/2026-05-05-mermaid-architettura\|Diagramma Architettura Sistema — Mermaid]] — Diagramma Mermaid TO-BE; ⚠️ conflict: nodo "API Gateway" vs no-API-GW confermato da CSI

### Baseline AS-IS

- [[wiki/sources/2019-02-01-sfu-gestione-consensi-v1-7\|Studio Funzionale Gestione Consensi SoL V1.7 (AS-IS 2019)]] — Baseline funzionale AS-IS: 6 CDU originali, stack legacy
- [[wiki/sources/2019-06-01-webservice-consenso-regionale-v03\|Specifica WebService ConsensoRegionaleAziendale v03 (AS-IS)]] — 5 servizi SOAP AS-IS: SRV-01÷05, WSDL outbound, sicurezza X509
- [[wiki/sources/2019-04-08-dizionario-dati-as-is\|Modello Dizionario Dati AS-IS (2019)]] — 12 entità AS-IS (2 business + 1 log + 9 dominio); cons_s_consenso non popolata
- [[wiki/sources/2019-03-20-acc-del-cdu-01-servizi-acquisizione\|ACC-DEL-CDU-01 Servizi Acquisizione Consensi (AS-IS)]] — Specifiche SOAP inbound: DA01-DA03, upsert logic, 18 codici errore

---

## Analyses

- [[wiki/analyses/valutazione-qualita-srs-consensi\|Valutazione Qualità SRS — Gestione Consensi]] — Analisi critica SRS: 7 punti forza, 5 rischi (3 critici + 2 moderati), checklist pre-sprint 12 item
- [[wiki/analyses/analysis-gap-as-is-to-be\|Analisi Gap AS-IS → TO-BE — Gestione Consensi]] — Delta 6→17 CDU, 12→26 entità DB, 3 cambiamenti semantici critici per integrazione SIA ASR
- [[wiki/analyses/analysis-2026-05-06-checklist-avvio-progetto\|Checklist Avvio Progetto — Gestione Consensi]] — Quadro completo per sviluppatori: 6 sezioni (confermato/da chiedere CSI/da produrre/rischi/azioni giorno 1)
- `openapi-cdu-15-16-v0.1.yaml` — Specifica OpenAPI 3.0 DRAFT: CDU-15 (stato consenso) + CDU-16 (configurazione ente), Bearer JWT, RFC 7807, 5 TBD da confermare con CSI (file YAML in `wiki/analyses/`, vedi [[wiki/analyses/analysis-2026-05-06-openapi-cdu-15-16\|analysis-2026-05-06-openapi-cdu-15-16]])
- [[wiki/analyses/analysis-2026-05-06-openapi-cdu-15-16\|analysis-2026-05-06-openapi-cdu-15-16]] — Descrizione OpenAPI CDU-15/16: 19 punti consolidati, 5 TBD CSI, 8 attività interne, cronologia versioni, prossimi passi
- [[wiki/analyses/analysis-2026-05-14-tr34-alternativa-batch-03\|analysis-2026-05-14-tr34-alternativa-batch-03]] — Risposta TR34: tracker decisionale CDU-17 PULL (centro stella); ✅ **confermato 20/07/2026** (storico della proposta iniziale)
- [[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|analysis-2026-05-14-risposte-mf-srs-v3]] — Tracker master 69 commenti SRS v3 lavorazione + 30 risposte MF; mappatura per tema (A–J) e propagazione wiki
- [[wiki/analyses/analysis-2026-05-14-punti-aperti-csi\|Punti Aperti da Chiedere a CSI Piemonte — Tracker Unificato]] — Consolidato 38 punti aperti aggregati da 8 pagine, raggruppati per area (auth, security, pull, openapi, batch, integrazioni, infra, gov) e sprint
- [[wiki/analyses/analysis-2026-05-27-punti-aperti-spiegati\|Punti Aperti — Spiegati in Modo Semplice]] — Versione in linguaggio piano del tracker: per ogni punto cosa significa, perché blocca, come si chiude
- [[wiki/analyses/analysis-2026-06-10-riassunto-presentazione-cliente\|Riassunto Esecutivo — Presentazione Progetto Gestione Consensi]] — Sintesi completa per presentazione cliente: contesto, funzionalità, architettura, sicurezza, batch, migrazione, pianificazione, ADR, punti aperti
- [[wiki/analyses/conspref-dmp-tracker\|CONSPREF-DMP — Tracker Piano Migrazione Dati]] — Tracker stato Data Migration Plan PG9→PG18 (target aggiornato da PG17); 7 sezioni con owner.
- [[wiki/analyses/analysis-2026-08-05-stato-scaduto-comunicazione-sia\|Stato SCADUTO — semantica e comunicazione asincrona ai SIA/ASR (BAT-03)]] — Stato consolidato SCADUTO (wiki + SRS v6), omonimia AS-IS/TO-BE, buco canale async (no notifica, fuori snapshot CDU-17), 3 opzioni + checklist call CSI
- [[wiki/analyses/analysis-2026-08-05-stato-scaduto-spiegato-semplice\|Stato SCADUTO — Spiegato in Modo Semplice]] — Versione in linguaggio piano per il cliente non tecnico: analogia del modulo firmato, tranello del cambio di significato, 3 strade con pro/contro, 8 domande per la call


---

## Architecture Decision Records

Registry decisioni architetturali in `docs/adr/`. Lifecycle gestito (proposed/accepted/superseded), cross-link bidirezionale con concept/analysis pages. 21 ADR (19 materializzati il 2026-05-19 + ADR-020/ADR-021 il 06/08/2026 — ADR-020 supersede ADR-017, ADR-021 supersede ADR-011 e ADR-019).

Indice completo: [docs/adr/README.md](wiki/docs/adr/README.md)

| ADR | Titolo | Status |
|---|---|---|
| [ADR-001](ADR-001-stack-tecnologico.md) | Stack tecnologico (Spring Boot 3 + Java 17 + Angular 19 + PG18) | accepted |
| [ADR-002](ADR-002-piattaforma-ecaas.md) | Piattaforma ECaaS Kubernetes Nivola + vincoli | **superseded** |
| [ADR-003](ADR-003-dbaas-nivola.md) | DBaaS Nivola esterno al namespace | accepted |
| [ADR-004](ADR-004-no-api-gateway.md) | No API Gateway — sicurezza applicativa Spring Security | accepted |
| [ADR-005](ADR-005-sicurezza-cdu-15-16.md) | Sicurezza CDU-15/16 OAuth2 CC + JWT + per-ente | accepted |
| [ADR-006](ADR-006-batch-03-pull-cdu-17.md) | BATCH-03 push → CDU-17 PULL centro stella | **accepted** |
| [ADR-007](ADR-007-batch-01-5min-skip-locked.md) | BATCH-01 5 min con SKIP LOCKED | accepted |
| [ADR-008](ADR-008-ssot-form-renderer.md) | SSoT Form Renderer (Citt + Op) — nota scope: solo Op | accepted |
| [ADR-009](ADR-009-eliminazione-sistemats.md) | Eliminazione SistemaTS | accepted |
| [ADR-010](ADR-010-cdu-01-split.md) | Split CDU-01 (01a Op + 01b Citt) — nota scope: solo 01a | accepted |
| [ADR-011](ADR-011-merge-cdu-04-05-cittadino.md) | Merge CDU-04/05 cittadino — pulsante unico | **superseded** |
| [ADR-012](ADR-012-notificatore-deleghe-post-completato.md) | Notificatore di Deleghe ≠ UNP, post-COMPLETATO | accepted |
| [ADR-013](ADR-013-migrazione-pg-dump-restore.md) | Migrazione PG9 → PG18 dump/restore | accepted |
| [ADR-014](ADR-014-apache-cxf-soap-client.md) | Apache CXF client SOAP | accepted |
| [ADR-015](ADR-015-storicizzazione-immutabile.md) | Storicizzazione immutabile | accepted |
| [ADR-016](ADR-016-scaduto-async-batch-02.md) | SCADUTO async via BATCH-02 | accepted |
| [ADR-017](ADR-017-lis-terzo-canale.md) | LIS terzo canale acquisizione | **superseded** |
| [ADR-018](ADR-018-rfc-7807-error-response.md) | RFC 7807 error response | accepted |
| [ADR-019](ADR-019-cdu-06-pdf-scope-ridotto.md) | CDU-06 PDF scope ridotto | **superseded** |
| [ADR-020](ADR-020-lis-integrazione-be-esistente.md) | LIS/RIS integrazione BE esistente (supersede ADR-017) | accepted |
| [ADR-021](ADR-021-perimetro-solo-operatore.md) | Perimetro progetto: solo Webapp Operatore (supersede ADR-011, ADR-019) | accepted |

