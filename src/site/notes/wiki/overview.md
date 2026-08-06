---
{"dg-publish":true,"permalink":"/wiki/overview/","title":"Overview","tags":["gestione-consensi","sanita-piemonte","srs","exprivia"],"dg-note-properties":{"title":"Overview","aliases":["Overview"],"type":"overview","tags":["gestione-consensi","sanita-piemonte","srs","exprivia"],"created":"2026-05-05","updated":"2026-08-06","sources":["2026-03-02-conspref-srs-v1-revised","2026-03-02-appunti-e-pianificazione","2026-03-02-domande-srs-csi-v02","2023-09-01-conspref-srs-01-v03","2019-02-01-sfu-gestione-consensi-v1-7","2026-03-12-pile-tecnologiche-csi"]}}
---


# Wiki Overview — Progetto Gestione Consensi

**Dominio:** Rifacimento applicativo Gestione Consensi, Sanità Elettronica Regione Piemonte
**Fornitore:** [[wiki/entities/exprivia\|Exprivia S.p.A.]] per [[wiki/entities/csi-piemonte\|CSI Piemonte]] / Regione Piemonte
**Fase attuale:** Analisi/SRS in bozza, pre-sviluppo

> 🔴 **Perimetro progetto (call CSI 06/08/2026, [[wiki/docs/adr/ADR-021-perimetro-solo-operatore\|ADR-021]]):** il progetto sviluppa **solo la Webapp Operatore**. La Webapp Cittadino esiste ma è fuori dal perimetro di sviluppo di questo progetto — non un suo deliverable.

---

## Sintesi del progetto

[[wiki/entities/exprivia\|Exprivia S.p.A.]] sta analizzando il rifacimento completo dell'applicativo [[wiki/concepts/gestione-consensi-applicativo\|Gestione Consensi - Applicativo]] della Regione Piemonte. Il sistema gestisce i consensi sanitari dei cittadini piemontesi (tre livelli: nazionale, regionale, aziendale) attraverso 17 casi d'uso, 2 processi batch (più il CDU-17 PULL) e 6 sistemi esterni.

Il documento centrale è **CONSPREF-SRS-V1.0**, redatto da Marco Forneris/Exprivia il 02/03/2026, che specifica il sistema TO-BE con stack tecnologico Angular 19 + Spring Boot 3 + PostgreSQL 18 (target aggiornato da PG17, call CSI 06/08/2026) su infrastruttura cloud [[wiki/concepts/architettura-iaas\|Architettura IaaS]] (IaaS Nivola, provisioning CSI). Ultima revisione recepita: **`CONSPREF-SRS-V1.0-revised_v6`** (rev 1.1, 15/07/2026 — GASP Shibboleth SP, sicurezza API Manager APIMBBONE, DBaaS DEV/pre-prod, toolchain ADA/Chef, §7.4 Manutenzione ASR). Chiarimenti sicurezza/integrazioni recepiti dalla **call CSI 20/07/2026** (header CF+`codice_ente`, Q1–Q6 delegati ad APIMBBONE, AURA/Deleghe chiusi). In questa fase gli ambienti provisionati sono **DEV e pre-produzione** (PROD rinviato per costi); il modello IaaS resta valido per tutti gli ambienti.

---

## Stato dell'analisi

**Giudizio:** alta qualità per un documento in bozza. Il lavoro è stato fatto bene. Le lacune principali sono:
1. **CONSPREF-DMP non formalizzato** — rischio critico per Fase 6 migrazione PG9→PG18 (target aggiornato da PG17, 06/08/2026); redazione in carico a **CSI Piemonte** (confermato 16/07/2026 — GOV-03 chiuso)
2. ✅ ~~Protocollo GASP Salute (OIDC vs SAML2) — aperto, blocca CDU-01~~ → **SAML2 confermato** da CSI (verbale 11/06/2026) — CDU-01 può procedere alla progettazione
3. **OpenAPI CDU-15/16** — [[wiki/analyses/analysis-2026-05-06-openapi-cdu-15-16\|v0.1-DRAFT prodotta]]; 5 TBD da confermare con CSI prima di condividere con [[wiki/concepts/sistemi-esterni-integrati\|SIA ASR]]

Vedi [[wiki/analyses/valutazione-qualita-srs-consensi\|Valutazione Qualità SRS — Gestione Consensi]] per analisi completa.

---

## Architettura del sistema (sintesi)

```
Cittadino (SPID/CIE) → GASP Salute → Angular SPA → Spring Boot 3 → PostgreSQL 18 (DBaaS)   ❌ OUT scope (ADR-021, 06/08/2026)
Operatore (RUPAR/IRIDE) → PUA → Angular SPA → Spring Boot 3                                 ✅ IN scope — unico deliverable

Spring Boot → AURA (SOAP), Deleghe (SOAP), UNP (REST), SIA-ASR (SOAP+REST)
```

---

## Temi chiave

### [[wiki/concepts/ciclo-vita-consenso\|Ciclo di Vita del Consenso]]
5 stati: NON_ESPRESSO → ATTIVO/NEGATO → SCADUTO/ANNULLATO. No sovrascrittura. Notifica asincrona via BATCH-01 ogni 5 minuti.

### Migrazione PostgreSQL 9 → 18
9 major release di salto (target aggiornato da PG17 a PG18, call CSI 06/08/2026). Strategia dump/restore. CONSPREF-DMP da redigere (in carico a CSI Piemonte, 16/07/2026). Rischi documentati: autenticazione (md5→scram), tipi deprecati, comportamento timestamp.

### Sicurezza
**AS-IS:** sicurezza applicativa Spring Security diretta (no API Gateway esterno). **TO-BE (nuovi fruitori):** doppia esposizione — certificati firmati (AS-IS invariato) + **API Manager CSI Piemonte (APIMBBONE)** (verbale 11/06/2026; modello confermato 07/2026). Credenziali gestite lato infrastruttura IaaS CSI. OWASP Top 10. CF mascherato nei log.
Per CDU-15/16/17 (servizi REST verso SIA ASR): token **OAuth2 `client_credentials` emesso e validato dall'API Manager APIMBBONE** (Key Manager + Gateway; rate limiting a carico APIM, non più `bucket4j`/JWKS lato backend). ✅ **Call CSI 20/07/2026:** il Gateway APIM **inoltra al backend CF (da Shibboleth) + `codice_ente`** in header/claim → l'isolamento per ente si basa sul `codice_ente` del gateway + WHERE clause repository (Livello C); la tabella `cons_t_client_ente` **esce dallo scope V1.0** (estensione futura multi-ente). TTL/refresh, scope, onboarding e revoca credenziali **delegati ad APIMBBONE**. Resta a nostro carico la produzione dello **swagger** per la sottoscrizione. Dettaglio: [[wiki/concepts/sicurezza-cdu-15-16\|Sicurezza CDU-15-16 — Modello Autorizzazione per Ente]].

---

## Documenti ingestionati (12/12)

| Documento | Status |
|---|---|
| CONSPREF-SRS-V1.0 revised v2 | ✅ |
| Sommario SRS Consensi (TOC strutturale) | ✅ |
| Appunti sistema + Pianificazione | ✅ |
| Domande SRS — revisione CSI V02 | ✅ |
| CONSPREF-SRS-01-V03 | ✅ |
| SFU Gestione Consensi V1.7 AS-IS | ✅ |
| Pile Tecnologiche CSI (mar 2026) | ✅ |
| Linea Guida Fornitori v1.0.1 cloud | ✅ |
| WebService ConsensoRegionaleAziendale v03 | ✅ |
| CSI Modello Dizionario Dati | ✅ |
| ACC-DEL-CDU-01-V01 Servizi Acquisizione | ✅ |
| Mermaid.txt (diagramma architettura) | ✅ |

---

## Evolution

- **2026-05-05** — Primo ingest: 6 documenti. Struttura wiki creata. Valutazione SRS.
- **2026-05-05** — Completamento corpus: 4 documenti rimanenti ingestionati. 2 nuovi rischi identificati: ambiguità BATCH-01/WSDL e differenza AS-IS/TO-BE stato SCADUTO.
- **2026-05-05** — Ingest Sommario SRS (TOC strutturale). Confermato: 25 tabelle TO-BE, 10 [PROPOSTA] §8.4. Index rielaborato con quick reference rischi e raggruppamento per categoria.
- **2026-05-05** — **Lint completo:** 6 nuove pagine create ([[wiki/concepts/gasp-salute\|GASP Salute]], [[wiki/concepts/batch-processes\|Processi Batch — BATCH-01, BATCH-02, BATCH-03]], [[wiki/concepts/sistemi-esterni-integrati\|Sistemi Esterni Integrati]], [[wiki/entities/regione-piemonte\|Regione Piemonte]], [[wiki/sources/2026-05-05-mermaid-architettura\|Diagramma Architettura Sistema — Mermaid]], [[wiki/analyses/analysis-gap-as-is-to-be\|Analisi Gap AS-IS → TO-BE — Gestione Consensi]]). Cross-ref densificate su 5 pagine esistenti. Filename ACC-DEL corretto nel log. Totale wiki: 24 pagine.
- **2026-05-14** — Risposta tecnica al commento cliente **TR30** (SRS v3 §6.16): creato concept [[wiki/concepts/sicurezza-cdu-15-16\|Sicurezza CDU-15-16 — Modello Autorizzazione per Ente]] con modello a 3 livelli (JWT + tabella mapping + WHERE clause), pseudocodice filter Spring Security, schema tabella `cons_t_client_ente`, 6 gap SRS e 6 punti aperti da CSI. Confermato esplicitamente: **no API Manager/Gateway** in path. Corpus 27→28 pagine.
- **2026-05-14** — Risposta tecnica al commento cliente **TR34** (SRS v3 §7.3 BATCH-03): proposta sostituzione BATCH-03 push con [[wiki/concepts/alternativa-batch-03-pull\|Alternativa BATCH-03 — PULL CDU-17 (centro stella)]] — endpoint REST paginato cursor-based, hub-and-spoke pull, zero push burden sul sistema regionale, zero downtime. Riusa interamente security CDU-15/16. BATCH-03 da rimuovere dal SRS. Variante export-with-downtime documentata come fallback. Status: PROPOSTA awaiting CSI. Corpus 28→30 pagine.
- **2026-05-14** — **Propagazione 30 risposte MF** da SRS v3 lavorazione (69 commenti). Nuovo concept [[wiki/concepts/composizione-dinamica-form-consenso\|Composizione Dinamica Form Consenso — Single Source of Truth]] (pattern SSoT Form Renderer unico Citt+Op). Tracker master [[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|analysis-2026-05-14-risposte-mf-srs-v3]]. Aggiornati 5 concept + 1 analysis (gap AS-IS→TO-BE) + 1 source. Eliminato SistemaTS dal corpus integrazioni. BATCH-01 scheduling 5min+SKIP LOCKED, Notificatore di Deleghe distinto da UNP, LIS 3° canale acquisizione. Aperto: SC67 (storicizzazione INSERT BATCH-02). Corpus 30→31 pagine.
- **2026-05-14** — **Tracker unificato punti aperti CSI** ([[wiki/analyses/analysis-2026-05-14-punti-aperti-csi\|Punti Aperti da Chiedere a CSI Piemonte — Tracker Unificato]]): 38 punti consolidati da 8 pagine, raggruppati per area (Identità, Sicurezza CDU-15/16, Pull CDU-17, OpenAPI, Batch, Integrazioni, Infra, Gov) e prioritizzati per sprint. 10 punti Giorno 1 / Sprint 0 critico. Corpus 31→32 pagine.
- **2026-05-14** — **Lint wiki:** zero dead-links, zero orfani. Fix: rinomina page "Pile Tecnologiche CSI Piemonte" (era "(marzo 2026)") — chiude 6 riferimenti dead. Aggiornati frontmatter `related` in source SRS-v1-revised (3 link). Q&A CSI #N nel checklist convertiti in alias verso [[wiki/sources/2026-03-02-domande-srs-csi-v02\|Domande SRS Consensi — Revisione CSI V02]]. Rimosso duplicato Concept/Source nell'index. Corpus ricalibrato 32→31 (errore di conteggio precedente).
- **2026-06-17-18** — **Audit post-verbale 11/06/2026 (CSI/Exprivia):** 29 fix applicati. IaaS Nivola confermato per tutti gli ambienti DEV/TEST/PROD — superati tutti i riferimenti ECaaS/Kubernetes (ADR-002 → superseded). GASP Salute SAML2 confermato (ID-01 chiuso; CDU-01 può procedere). Sicurezza: doppia esposizione AS-IS (certificati) + TO-BE (API Manager CSI + JWS) per nuovi fruitori. Batch manutenzione ASR documentato. Gestione Deleghe via API-Piemonte/`getDelegantiService` formalizzata.
- **2026-07-13** — **Recepimento risposte CSI (email 07/2026), 4 punti:** (1) **GASP** — metadata Service Provider ricevuti; integrazione via **Shibboleth SP** (non `spring-security-saml2`). (2) **Sicurezza** — token servizi SIA gestito dall'**API Manager CSI (APIMBBONE)** OAuth2 `client_credentials` (Key Manager + Gateway); rate limiting a carico APIM; **swagger** prerequisito; superati Authorization Server/JWKS/`bucket4j`/"token JWS". (3) **Database** — provisioning DBaaS **DEV + pre-prod** (PROD rinviato); ribaltamento dati da **TEST PG 9.6** su DEV. (4) **IaaS** — toolchain **GitLab/Jenkins/SonarQube/Artifactory/ADA-Chef/ASGARD**; deploy via **ADA/Chef** (no Helm/K8s); residui: ingress/TLS + segreti. Prodotto deliverable **`CONSPREF-SRS-V1.0-revised_v5`** (.md/.docx) con S-01…S-04 applicate. Concept aggiornati: gasp-salute, sicurezza-cdu-15-16, architettura-iaas, migrazione-postgres, stack-tecnologico; tracker INF/SEC/ID; overview.
- **2026-07-20** — **Recepimento call CSI (chiarimenti punti aperti):** (1) **Sicurezza API (Q1–Q6)** — il Gateway APIMBBONE inoltra sempre **CF (da Shibboleth) + `codice_ente`** in header/claim; TTL/refresh, scope, onboarding SIA (evento raro → CR) e revoca credenziali **delegati ad APIMBBONE**; tabella `cons_t_client_ente` fuori scope V1.0 (estensione futura), resta il Livello C. (2) **AURA (INT-01/ID-03)** — nessun servizio nuovo, stessi WSDL nei properties, credenziali IRIS DEV incluse → chiuso. (3) **Gestione Deleghe (INT-02)** — già integrato, nulla da fare. (4) **BATCH-01 (BAT-01)** — SRV-03/SRV-04 confermati, "solo da svecchiare". (5) **GOV-02** — deroga V03 `online`/`annulla_consensi` approvata (sorgente unica `cons_d_informativa`). (6) **SC67** — CSI non aveva capito la domanda: riformulata a scenario (flag da informativa scaduta vs nuova). Aperti/differiti: **BAT-03** (Stato SCADUTO, call dedicata), Manutenzione ASR §7.4, frequenza BATCH-02, lista ASR, **swagger CDU-15/16/17** (prerequisito APIM), **CDU-17** (rielaborato il 20/07 — vedi entry successiva). Concept aggiornati: sicurezza-cdu-15-16, batch-processes, gasp-salute, sistemi-esterni-integrati; tracker punti-aperti + spiegati; overview; ADR-005/ADR-016.
- **2026-07-20** — **Rifacimento CDU-17 (call CSI):** il committente ha fornito il rifacimento completo del caso d'uso. Attore = operatore **Back Office** (web app da PUA). **Variante B (watermark) eliminata** → blocco acquisizioni **obbligatorio**. **Nuovo passo 5:** il SIA notifica alla webapp `COMPLETATO` + dati ultimo invio (canalità PULL-02). **PULL-02 deciso:** email **e/o** webhook configurabile (webhook → il SIA espone un REST, contratto fornito da CSI). **Servizi endpoint CRUD** (CDU-14) esposti alle aziende via **API Manager**; web app diretta (no APIM). **Nuovo scenario di manutenzione endpoint** (stato `IN_MANUTENZIONE`, blocco acquisizione + notifica webapp blocco/ripresa). Sicurezza allineata a 07/2026 (`codice_ente` dal Gateway, `cons_t_client_ente` fuori scope V1.0). **ADR-006 → accepted** (PULL-01/02/05/06/08 chiusi). Prodotti/aggiornati: `CDU-17_diagramma-sequenza` (+ nuovo `Manutenzione-endpoint_diagramma-sequenza`, con SVG/PDF). Allineati: alternativa-batch-03-pull, batch-processes, sistemi-esterni-integrati, tracker punti-aperti + spiegati, ADR-006/005, index. Residuo attivo: **swagger (OpenAPI) CDU-17**.
- **2026-08-06** — **Recepimento call CSI:** (1) **Migrazione dati** — target aggiornato **PG9→PG18** (era PG17; CSI ha nel frattempo migrato il DBaaS). Nessun cambio di strategia (dump/restore invariato). (2) **LIS/RIS (chiude INT-03)** — CSI ha chiarito che **non esiste un terzo canale di acquisizione consensi**: LIS acquisisce tramite **integrazione BE già presente nel codice sorgente AS-IS**, da individuare/verificare e migrare al nuovo stack — non un nuovo canale UI. Creato **ADR-020** (supersede [[wiki/docs/adr/ADR-017-lis-terzo-canale\|ADR-017]]). Aggiornati: ADR-001/003/007/013/015/017/020, migrazione-postgres-9-17, stack-tecnologico-applicativo, architettura-iaas, batch-processes, gestione-consensi-applicativo, sistemi-esterni-integrati, composizione-dinamica-form-consenso, conspref-dmp-tracker, valutazione-qualita-srs-consensi, exprivia, tracker punti-aperti + spiegati, index, docs/adr/README. **Allineamento SRS rimandato**: da eseguire solo dopo conferma esplicita dell'utente.
- **2026-08-06** — **Correzione perimetro progetto (call CSI):** il cliente ha chiarito che il progetto **non copre 2 webapp ma 1 sola: la Webapp Operatore**. La Webapp Cittadino esiste ma è **fuori dal perimetro di sviluppo**. Creato **ADR-021** (supersede [[wiki/docs/adr/ADR-011-merge-cdu-04-05-cittadino\|ADR-011]] e [[wiki/docs/adr/ADR-019-cdu-06-pdf-scope-ridotto\|ADR-019]], entrambi esclusivamente su funzionalità cittadino); note di scope aggiunte a [[wiki/docs/adr/ADR-010-cdu-01-split\|ADR-010]] e [[wiki/docs/adr/ADR-008-ssot-form-renderer\|ADR-008]] (restano accepted, non superseded). CDU fuori scope: CDU-01b, CDU-02, CDU-03, CDU-04, CDU-06. Aggiornati: gestione-consensi-applicativo (canali, profili, CDU-01, funzionalità TO-BE), gasp-salute (banner fuori scope), composizione-dinamica-form-consenso (banner + gap SRS annotati), ciclo-vita-consenso (tabella logica CDU), index, docs/adr/README, tracker viventi (gap-as-is-to-be, valutazione-qualita-srs). **Allineamento SRS rimandato**: da eseguire solo dopo conferma esplicita dell'utente.

