---
{"dg-publish":true,"permalink":"/wiki/analyses/analysis-2026-05-06-checklist-avvio-progetto/","title":"Checklist Avvio Progetto — Gestione Consensi","tags":["checklist","avvio","sprint-0","cliente","csi-piemonte","exprivia","rischi","prerequisiti"],"dg-note-properties":{"title":"Checklist Avvio Progetto — Gestione Consensi","aliases":["Checklist Avvio Progetto — Gestione Consensi"],"type":"analysis","tags":["checklist","avvio","sprint-0","cliente","csi-piemonte","exprivia","rischi","prerequisiti"],"created":"2026-05-06","updated":"2026-06-17","sources":["2026-03-02-conspref-srs-v1-revised","2026-03-02-domande-srs-csi-v02","2026-03-02-appunti-e-pianificazione","2023-09-01-conspref-srs-01-v03","2019-06-01-webservice-consenso-regionale-v03","2019-03-20-acc-del-cdu-01-servizi-acquisizione"],"related":["[[analysis-gap-as-is-to-be|Analisi Gap AS-IS → TO-BE — Gestione Consensi]]","[[Architettura IaaS]]","[[CSI Piemonte]]","[[wiki/entities/exprivia\|Exprivia S.p.A.]]","[[wiki/concepts/batch-processes\|Processi Batch — BATCH-01, BATCH-02, BATCH-03]]","[[Sistemi Esterni Integrati]]","[[GASP Salute]]"]}}
---


# Checklist Avvio Progetto — Gestione Consensi

**Scopo:** Quadro completo dei punti cardine per far partire il progetto. Distingue ciò che è già confermato da ciò che deve essere richiesto a [[wiki/entities/csi-piemonte\|CSI Piemonte]] o prodotto internamente da [[wiki/entities/exprivia\|Exprivia S.p.A.]].

**Fonti:** SRS V1.0 bozza v2, Q&A CSI V02, appunti, pianificazione, spec WebService v03, ACC-DEL-CDU-01.

---

## SEZIONE A — Già confermato ✅

Punti validati dalla Q&A CSI V02 o esplicitamente documentati nell'SRS e negli appunti tecnici.

### A1. Stack tecnologico

| Componente              | Versione                                   | Fonte conferma |
| ----------------------- | ------------------------------------------ | -------------- |
| Frontend                | Angular 19.x                               | Q&A CSI #2     |
| UI Component Library    | QUASAR CSI                                 | Q&A CSI #3     |
| Backend                 | Spring Boot 3.4.10+ / Java 17              | Q&A CSI #5     |
| Database                | PostgreSQL 17 via DBaaS Nivola             | Q&A CSI #10    |
| CI/CD                   | GitLab + Jenkins + SonarQube + Helm GitOps | [[wiki/sources/2026-03-02-appunti-e-pianificazione\|appunti]] |
| Runtime                 | **IaaS Nivola** (DEV/TEST/PROD) — provisioning CSI | Verbale 11/06/2026 ⚠️ |
| SOAP client             | Apache CXF (necessario per SIA ASR AS-IS)  | Q&A CSI #9     |
| Error response standard | RFC 7807                                   | SRS §4.x       |

### A2. Integrazioni esterne — protocolli confermati

| Sistema                     | Protocollo           | Auth                             | Stato                              |
| --------------------------- | -------------------- | -------------------------------- | ---------------------------------- |
| AURA                        | SOAP                 | WS-Security UsernameToken (IRIS) | ✅ confermato Q&A #7                |
| Gestione Deleghe            | SOAP                 | OAuth2 Client Credentials        | ✅ confermato Q&A #7                |
| Notificatore UNP            | REST                 | Token UNP                        | ✅ confermato Q&A #8                |
| SIA ASR (outbound BATCH)    | SOAP AS-IS invariato | —                                | ✅ confermato Q&A #9                |
| SIA ASR (inbound CDU-15/16) | REST OpenAPI 3.x     | Bearer JWT                       | ✅ architettura, spec ❌ da produrre |
| [[wiki/concepts/gasp-salute\|GASP Salute]] | **SAML2**            | SPID/CIE                         | ✅ **confermato** — verbale 11/06/2026; metadata XML IdP da acquisire Sprint 0 |

### A3. Architettura applicativa

- No API Gateway CSI — integrazione diretta, sicurezza gestita da Spring Security ([[wiki/sources/2026-03-02-domande-srs-csi-v02\|Q&A CSI #6]])
- Skeleton progetto in carico a **Exprivia** (IaaS); confronto su POM con CSI — verbale 11/06/2026. ~~CSI fornisce automation per struttura + pipeline + Helm~~ (superato)
- DBaaS Nivola per PostgreSQL — non PG nel pod K8s ([[wiki/sources/2026-03-02-domande-srs-csi-v02\|Q&A CSI #10]])
- Credenziali gestite lato infrastruttura **IaaS CSI** — nessun segreto nel codice sorgente (no K8s Secret: ambiente IaaS non Kubernetes)
- ~~Scheletro progetto generato da automation CSI~~ → Skeleton in carico a **Exprivia** (IaaS, non automation CSI) — vedi B4 ✅

### A4. Funzionalità — scope confermato

- **16 CDU** (tutti documentati con scenario principale, varianti, algoritmi, campi validati)
- **2 processi batch attivi + 1 sostituito** ([[wiki/concepts/batch-processes\|Processi Batch — BATCH-01, BATCH-02, BATCH-03]]): BATCH-01 ogni 5 min, BATCH-02 su scadenza. BATCH-03 push **sostituito** da PULL CDU-17 (vedi [[wiki/concepts/alternativa-batch-03-pull\|Alternativa BATCH-03 — PULL CDU-17 (centro stella)]] e [ADR-006](ADR-006-batch-03-pull-cdu-17.md) — **proposed**, attende sign-off CSI)
- **25 tabelle TO-BE** (§8.3.1–8.3.25 dell'SRS)
- **No sovrascrittura consensi** — storicizzazione immutabile via `cons_s_consenso`
- **CDU-06 PDF** è funzionalità nuova (non presente AS-IS) — firma eIDAS **non richiesta** ([[wiki/sources/2026-03-02-domande-srs-csi-v02\|Q&A CSI #13]])
- WSDL outbound verso SIA ASR: namespace `http://consprefbe.csi.it/` invariato

### A5. Sicurezza — requisiti confermati

- OWASP Top 10 + CSRF su POST/PUT
- TLS 1.2 minimo
- Codice Fiscale mascherato nei log
- Conformità Linee Guida Fornitori Cloud Native v1.0.1 — **SRS §3.5 pienamente conforme** (verificato su documento originale)

### A6. Migrazione PG9 → PG17

- Strategia: **dump & restore logico** (`pg_dump`/`pg_restore`) — no `pg_upgrade` diretto (8 major release di salto)
- Punti critici già identificati: autenticazione md5→scram-sha-256, SERIAL deprecato, `default_with_oids` rimosso, operatori JSON/JSONB, sequenze orfane, `AT TIME ZONE` in PG15
- Piano cutover PROD: stop app → dump PG9 → restore PG17 → aggiorna K8s Secret → restart → smoke test → rollback standby 48h

---

## SEZIONE B — Da chiedere a CSI Piemonte ❌

### 🔴 CRITICO — Bloccante per Sprint 1

| #   | Richiesta                                                                                                              | Impatto se non ottenuto                          | Urgenza                 |
| --- | ---------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ | ----------------------- |
| B1  | ~~Documentazione tecnica GASP Salute — protocollo (OIDC vs SAML2)~~ ✅ **CHIUSO** — Protocollo **SAML2** confermato (verbale 11/06/2026). Da acquisire: documentazione tecnica GASP (endpoint, metadata XML). | CDU-01 può procedere | ✅ Sprint 0 |
| B2  | **Provisioning DBaaS Nivola DEV** — scheda provisioning standard                                                       | Impossibile fare Sprint 2 (DDL TO-BE)            | **Giorno 1 Sprint 0**   |
| B3  | **Provisioning DBaaS Nivola PROD**                                                                                     | Fase 6 migrazione bloccata                       | **Avviare in Sprint 0** |
| B4  | ~~Accesso automation CSI — skeleton progetto~~ ✅ **CHIUSO** — Skeleton in carico a **Exprivia** (IaaS). Confronto su POM con CSI (verbale 11/06/2026). | — | ✅ |

### 🟠 ALTO — Bloccante per Sprint 2-3

| #   | Richiesta                                                                                       | Impatto                                       | Urgenza  |
| --- | ----------------------------------------------------------------------------------------------- | --------------------------------------------- | -------- |
| B5  | **WSDL AURA** — lista completa servizi da chiamare per CDU-07/08 (ricerca paziente + operatori) | Client SOAP AURA non implementabile           | Sprint 0 |
| B6  | **WSDL Gestione Deleghe**                                                                       | Client SOAP Deleghe non implementabile        | Sprint 0 |
| B7  | **Credenziali IRIS** per autenticazione AURA (ambiente DEV)                                     | Test integrazione AURA impossibile            | Sprint 1 |
| B8  | **Accesso repo QUASAR CSI** (componenti UI)                                                     | Frontend Sprint 2+ senza componenti ufficiali | Sprint 1 |
| B9  | **Registrazione app PUA** — 2 profili (Operatore, Amministratore)                               | Test autenticazione operatori impossibile     | Sprint 2 |

### 🟡 MODERATO — Necessario prima UAT

| #   | Richiesta                                                                                                                             | Impatto                                                                                                        | Urgenza           |
| --- | ------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- | ----------------- |
| B10 | **Conferma operazione WSDL per BATCH-01** — SRV-01 (AcquisizioneConsenso, inbound) o SRV-03 (NotificaAcquisizioneConsenso, outbound)? | ⚠️ Rischio implementativo grave se sbagliato — vedi [[wiki/analyses/valutazione-qualita-srs-consensi\|Valutazione Qualità SRS — Gestione Consensi]] §Rischio 4 | Sprint 1          |
| B11 | **Lista ASR coinvolte nel TO-BE** + referenti tecnici                                                                                 | Impossibile coordinarsi su semantica SCADUTO e su recepimento OpenAPI CDU-15/16                                | Sprint 2          |
| B12 | **Approvazione formale SRS V1.0 bozza v2** da CSI                                                                                     | Il team lavora su bozza non approvata — rischio richieste di modifica tardive                                  | Prima di Sprint 1 |
| B13 | **Validazione [PROPOSTA] nell'SRS** — in particolare: ALG02 BATCH-01 (gestione tentativi), CDU-06 PDF, 10 proposte §8.4               | Se CSI non valida, potremmo implementare funzionalità non volute                                               | Prima di Sprint 2 |
| B14 | **Il diagramma architetturale** (`Mermaid.txt`) è versione concordata con CSI o proposta Exprivia?                                    | Rischio disallineamento architetturale                                                                         | Sprint 0          |
| B15 | **SLA e NFR performance** — tempo risposta massimo CDU-02, throughput BATCH-01, disponibilità (99.x%)                                 | Collaudo senza criteri di accettazione definiti                                                                | Prima di Sprint 8 |

---

## SEZIONE C — Da produrre internamente (Exprivia) 🔧

| #   | Deliverable                                                                         | Responsabile             | Scadenza   | Note                                                               |
| --- | ----------------------------------------------------------------------------------- | ------------------------ | ---------- | ------------------------------------------------------------------ |
| C1  | **CONSPREF-DMP bozza v1** — Data Migration Plan PG9→PG17                            | ✅ **CSI Piemonte** (confermato 16/07/2026 — GOV-03 chiuso) | Sprint 0   | Senza questo, Fase 6 migrazione a rischio slittamento              |
| C2  | **Specifica OpenAPI 3.x CDU-15/16** — API stato consensi + configurazione per SIA   | Da assegnare             | Sprint 1-2 | **Non rimandare a Sprint 6** — le ASR devono recepirla in anticipo |
| C3  | **Audit DDL PG9** — `\d` su ogni tabella AS-IS su DB reale, focus `cons_s_consenso` | Team database            | Sprint 0   | Verificare struttura fisica vs DDL atteso TO-BE                    |

---

## SEZIONE D — Rischi aperti critici (sintesi)

| #   | Rischio                                                                                                              | Probabilità | Impatto  | Azione                                                 |
| --- | -------------------------------------------------------------------------------------------------------------------- | ----------- | -------- | ------------------------------------------------------ |
| R1  | ~~GASP Salute protocollo non definito → CDU-01 bloccato~~ ✅ **RISOLTO** — SAML2 confermato (verbale 11/06/2026) | — | — | — |
| R2  | DBaaS Nivola provisioning lento → Sprint 2 slittamento                                                               | Alta        | Alto     | B2/B3 — avviare giorno 1, latenza imprevedibile        |
| R3  | BATCH-01 usa operazione WSDL sbagliata (SRV-01 vs SRV-03)                                                            | Media       | Critico  | B10 — conferma scritta da CSI prima di implementare    |
| R4  | Semantica SCADUTO AS-IS ≠ TO-BE — SIA ASR non allineati                                                              | Media       | Alto     | B11 + C2 — documentare in OpenAPI + verificare con ASR |
| R5  | `cons_s_consenso` AS-IS struttura incompatibile TO-BE                                                                | Media       | Alto     | C3 — audit DDL Sprint 0                                |
| R6  | DMP non formalizzato → blocco Fase 6                                                                                 | Alta        | Alto     | C1 — responsabile confermato (CSI, 16/07/2026); sollecitare bozza v1 |
| R7  | ✅ OpenAPI CDU-15/16 — [[wiki/analyses/analysis-2026-05-06-openapi-cdu-15-16\|v0.1-DRAFT prodotta]] Sprint 0; 5 TBD CSI da chiudere | Alta        | Moderato | Condividere con ASR dopo conferma TBD Sprint 1-2       |

---

## SEZIONE E — Sequenza di azioni giorno 1

Ordine di priorità per il kick-off:

1. **Richiedere documentazione [[wiki/concepts/gasp-salute\|GASP Salute]]** a referente CSI (B1) — identifica il responsabile CSI oggi
2. **Avviare provisioning DBaaS Nivola DEV + PROD** (B2/B3) — compilare scheda standard e inviarla
3. **Richiedere WSDL AURA + Gestione Deleghe** (B5/B6)
4. **Richiedere accesso automation skeleton + repo QUASAR** (B4/B8)
5. **Richiedere conferma scritta operazione WSDL BATCH-01** (B10)
6. ~~Assegnare responsabile CONSPREF-DMP~~ ✅ (C1 — CSI Piemonte, confermato 16/07/2026)
7. ✅ **OpenAPI CDU-15/16 v0.1-DRAFT prodotta** — portare a Sprint 0, chiudere 5 TBD con CSI, condividere v0.2 con ASR entro Sprint 2
8. **Avviare audit DDL PG9** (C3) — accesso DB AS-IS necessario

---

## Note

- Tutti i punti della Sezione A sono vincolanti per l'SRS — qualsiasi cambio richiede variante formale
- Le 13 Q&A CSI V02 sono state incorporate nell'SRS — non rinegoziabili senza revisione formale
- Il tag `[PROPOSTA]` nell'SRS (§8.4) indica funzionalità non richieste dal committente — validazione CSI obbligatoria prima dell'implementazione
