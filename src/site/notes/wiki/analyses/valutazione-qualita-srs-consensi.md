---
{"dg-publish":true,"permalink":"/wiki/analyses/valutazione-qualita-srs-consensi/","title":"Valutazione Qualità SRS — Gestione Consensi","tags":["valutazione","qualita","srs","analisi-critica","gestione-consensi"],"dg-note-properties":{"title":"Valutazione Qualità SRS — Gestione Consensi","aliases":["Valutazione Qualità SRS — Gestione Consensi"],"type":"analysis","tags":["valutazione","qualita","srs","analisi-critica","gestione-consensi"],"created":"2026-05-05","updated":"2026-07-16","sources":["2026-03-02-conspref-srs-v1-revised","2026-03-02-appunti-e-pianificazione","2026-03-02-domande-srs-csi-v02","2023-09-01-conspref-srs-01-v03","2019-02-01-sfu-gestione-consensi-v1-7"],"related":["[[Gestione Consensi - Applicativo]]","[[ciclo-vita-consenso|Ciclo di Vita del Consenso]]","[[Architettura IaaS]]","[[exprivia|Exprivia S.p.A.]]","[[CSI Piemonte]]","[[analysis-2026-05-14-risposte-mf-srs-v3|Risposte MF — Revisione SRS v3 lavorazione (69 commenti)]]"]}}
---


# Valutazione Qualità SRS — Progetto Consensi

**Oggetto valutato:** CONSPREF-SRS-V1.0 (bozza v2) + appunti + pianificazione + Q&A CSI
**Data valutazione:** 2026-05-05
**Metodo:** analisi incrociata di tutti i documenti disponibili

---

---

## Punti di Forza ✅

### 1. Copertura funzionale completa (16 CDU)
L'AS-IS aveva 6 CDU (2019). Il TO-BE ne specifica 16, con espansione logica e ben motivata:
- Ogni CDU ha: obiettivo, precondizioni, scenario principale tabellare, varianti, algoritmi (ALG01/ALG02)
- Le "Informazioni trasmesse" sono specificate per ogni CDU con tipo campo, modalità (I/D/I/D), obbligatorietà, formato, validazione, messaggi di errore
- La distinzione CDU-04/CDU-05 (con/senza riaccettazione informativa) è precisa e corretta

### 2. Macchina a stati consenso 
- 5 stati ben definiti con semantica chiara
- Regola no-sovrascrittura documentata e implementativa (UPDATE + INSERT in cons_s_consenso)
- Distinzione SCADUTO (no notifica) vs ANNULLATO (notifica) motivata e corretta
- Trigger BATCH-01/02/03 collegati agli eventi applicativi

### 3. Architettura tecnologica — allineata alle pile CSI
- Spring Boot 3.4.10+ ✅ (CURRENT)
- PostgreSQL 17 ✅ (CURRENT da dic 2025)
- Java 17 ✅ (CURRENT)
- Angular 19 ✅ (CURRENT stack SPA v2.1.0)
- Infrastruttura IaaS Nivola ✅ (ECaaS/Kubernetes superato — verbale 11/06/2026)
- Gestione DBaaS Nivola: connessione sicura (segreti via meccanismo dell'ambiente IaaS, da concordare con CSI), HikariCP config, DB esterno all'ambiente applicativo

### 4. Integrazioni esterne
- AURA: SOAP + WS-Security UsernameToken (IRIS) — protocollo preciso
- Gestione Deleghe: SOAP + OAuth2 Client Credentials — preciso
- BATCH-01 verso SIA: SOAP AS-IS invariato confermato da CSI
- CDU-15/16 verso SIA: REST OpenAPI 3.x — contratto da produrre, responsabilità chiara
- RFC 7807 adottato per error response — standard corretto

### 5. Sicurezza
- No segreti nel codice sorgente (gestione segreti dell'ambiente IaaS)
- OWASP Top 10, CSRF su POST/PUT
- CF mascherato nei log
- Spring Security senza API Gateway (scelta confermata da CSI)
- TLS 1.2+
- Librerie open source (conformità Developers Italia)

### 6. Pianificazione dettagliata
- 8 fasi, 10 sprint, stima in Story Points (1SP=4h) per ogni attività
- Identificazione dipendenze critiche (DBaaS Nivola, AURA WSDL, GASP Salute doc)
- Piano migrazione PG9→PG17 con analisi rischi specifica e procedura cutover dettagliata

---

## Lacune e Rischi ⚠️

### RISCHIO CRITICO 1 — CONSPREF-DMP non formalizzato
**Problema:** Il Data Migration Plan PG9→PG17 non esiste ancora come documento formale.
**Impatto:** Sprint 0 include la bozza v1, ma se il provisioning DBaaS è lento (alta latenza), la Fase 6 Migrazione (sett. 13-16) rischia di slittare.
**Azione raccomandata:** Avviare la redazione DMP in parallelo all'SRS. ✅ Responsabile redazione confermato 16/07/2026: **CSI Piemonte** (GOV-03 chiuso); resta la produzione della bozza v1.

### ✅ RISCHIO CRITICO 2 — Protocollo GASP Salute — CHIUSO (verbale 11/06/2026)
**Protocollo confermato: SAML2.** Dipendenza Spring: `spring-security-saml2-service-provider`.
**Residuo:** Documentazione tecnica [[wiki/concepts/gasp-salute\|GASP Salute]] (metadata XML IdP, endpoint, entity ID) ancora da acquisire — richiedere in Sprint 0 giorno 1.
**Impatto residuo:** SAML2 richiede configurazione metadata XML lato SP e librerie dedicate. Più complesso di OIDC ma protocollo noto. CDU-01 può procedere alla progettazione.

### ✅ RISCHIO MODERATO 3 — OpenAPI CDU-15/16 — PARZIALMENTE MITIGATO
**Stato:** [[wiki/analyses/analysis-2026-05-06-openapi-cdu-15-16\|v0.1-DRAFT prodotta]] (2026-05-06) — 19 punti consolidati, 5 TBD da confermare con CSI.
**Impatto residuo:** I 5 TBD (URL base AS-IS, scope OAuth, rate-limiting, lista ASR, versioning) bloccano la condivisione con SIA ASR. UAT rimane a rischio se TBD non chiusi entro Sprint 1-2.
**Azione raccomandata:** Portare bozza v0.1 a riunione Sprint 0, chiudere TBD con CSI, condividere v0.2 con referenti ASR.

### ✅ LACUNA 4 — CHIUSA: Linee Guida Cloud Native verificate
**Risultato:** `Linea-Guida-Fornitori-v1.0.1_cloud.md` ingestionata. La sezione 3.5 dell'SRS è stata riscritta in chiave **IaaS Nivola** (ECaaS/Kubernetes superato — verbale 11/06/2026); i vincoli ECaaS della guida non si applicano e i dettagli operativi IaaS restano da concordare con CSI (INF-05).

### RISCHIO AGGIUNTO 4 — Ambiguità BATCH-01 / WebService spec
**Problema:** L'SRS §7.1 dice che BATCH-01 usa "operazione Acquisizione" per il contratto WSDL AS-IS. Ma dalla WebService spec v03, i servizi **uscenti** dal Regionale verso le ASR sono **SRV-03 NotificaAcquisizioneConsenso** e **SRV-04 NotificaRevocaConsenso** — non SRV-01 (AcquisizioneConsenso) che è inbound.
**Impatto:** Se il team implementa [[wiki/concepts/batch-processes\|BATCH-01]] chiamando SRV-01 invece di SRV-03, il contratto WSDL sarà sbagliato. Errore implementativo potenzialmente grave.
**Azione raccomandata:** Chiedere conferma a CSI: quale operazione WSDL usa BATCH-01? SRV-03 NotificaAcquisizioneConsenso o SRV-01 AcquisizioneConsenso?

### RISCHIO AGGIUNTO 5 — Differenza AS-IS vs TO-BE su stato SCADUTO
**Problema:** Nell'AS-IS ([[wiki/sources/2019-03-20-acc-del-cdu-01-servizi-acquisizione\|ACC-DEL-CDU-01 Servizi Acquisizione Consensi (AS-IS)]]), lo stato 'scaduto' viene impostato durante l'operazione di acquisizione stessa (se il consenso è già presente). Nel TO-BE invece SCADUTO è gestito esclusivamente da [[wiki/concepts/batch-processes\|BATCH-02]] alla scadenza dell'informativa. Sono due meccanismi distinti.
**Impatto:** Se il SIA_ASR si aspettano il comportamento AS-IS (scadenza durante acquisizione), il TO-BE potrebbe rompere l'integrazione.
**Azione:** Documentare la differenza nella specifica OpenAPI e verificare con i SIA ASR che siano allineati al nuovo comportamento. Vedi [[wiki/analyses/analysis-gap-as-is-to-be\|Analisi Gap AS-IS → TO-BE — Gestione Consensi]] §Cambiamenti semantici critici.

### ✅ LACUNA 5 — CHIUSA: WebService spec analizzata
**Risultato:** 5 servizi SOAP documentati. Contratto WSDL identificato. Ambiguità su BATCH-01 segnalata come rischio.

### ✅ LACUNA 6 — PARZIALMENTE CHIUSA: Dizionario dati AS-IS analizzato
**Risultato:** 12 entità AS-IS vs 25+ TO-BE. Il dizionario è scarno (nessun attributo visibile). La cons_s_consenso AS-IS risulta "non popolata" — rilevante per il TO-BE che la usa attivamente. L'audit DDL Sprint 0 rimane necessario sul DB reale.

### ✅ LACUNA 7 — CHIUSA: ACC-DEL-CDU-01 analizzato
**Risultato:** Specifiche SOAP inbound complete documentate. Trovata differenza semantica AS-IS/TO-BE su stato SCADUTO (segnalata come rischio).

### NOTA — Mancanza di sezione NFR performance/SLA
**Problema:** L'SRS specifica scalabilità e sicurezza ma non include SLA espliciti (es. tempo risposta massimo CDU-02, throughput BATCH-01, disponibilità 99.x%).
**Impatto:** Basso in questa fase, ma mancanza tipica da colmare prima del collaudo.

---

## Confronto AS-IS → TO-BE

Per analisi dettagliata del delta vedi [[wiki/analyses/analysis-gap-as-is-to-be\|Analisi Gap AS-IS → TO-BE — Gestione Consensi]].

| Dimensione | AS-IS (2019) | TO-BE (2026) | Evoluzione |
|---|---|---|---|
| CDU | 6 | 16 | +167% — ben giustificato |
| Download PDF | Non previsto | CDU-06 NUOVO | Aggiunta valore |
| API per SIA | Solo SOAP outbound | SOAP + REST CDU-15/16 | Apertura moderna |
| Stack DB | PostgreSQL 9 (RETIRED) | PostgreSQL 17 (CURRENT) | Obbligo tecnico |
| Architettura | Legacy | IaaS Nivola, microservizi Spring Boot | Allineamento CSI |
| Autenticazione | Non documentata | GASP Salute SPID/CIE | Modernizzazione |

---

## Checklist pre-avvio sprint di sviluppo

| Item | Stato | Note |
|---|---|---|
| SRS V1.0 approvata da CSI | ⏳ In revisione | Bozza v2 |
| Documentazione GASP Salute ricevuta | ❌ Da richiedere | Sprint 0 giorno 1 |
| WSDL AURA ricevuti | ❌ Da richiedere | Lista servizi da specificare |
| WSDL Gestione Deleghe ricevuti | ❌ Da richiedere | |
| DBaaS Nivola DEV provisioned | ❌ Da richiedere | Alta latenza |
| DBaaS Nivola PROD provisioned | ❌ Da richiedere | Alta latenza |
| Accesso repo QUASAR CSI | ❌ Da richiedere | Prerequisito UI |
| Registrazione app PUA (2 profili) | ❌ Da richiedere | |
| Credenziali IRIS per AURA | ❌ Da richiedere | |
| CONSPREF-DMP bozza v1 | ❌ Da produrre | Sprint 0 |
| Specifica OpenAPI CDU-15/16 bozza | ⚠️ [[wiki/analyses/analysis-2026-05-06-openapi-cdu-15-16\|v0.1-DRAFT prodotta]] — 5 TBD aperti | Condividere con ASR dopo TBD chiusi |
| Audit DDL PG9 → PG17 | ❌ Pianificato Sprint 0 | |

---

## Domande aperte per il team

1. Chi è il referente formale CSI per GASP Salute? Quando è disponibile la documentazione tecnica?
2. La specifica OpenAPI CDU-15/16 va condivisa con i SIA ASR: quali ASR sono coinvolte e chi è il referente?
3. ~~Il CONSPREF-DMP: chi è il responsabile formale?~~ ✅ Chiuso 16/07/2026: redazione in carico a CSI Piemonte (GOV-03)
4. I [PROPOSTA] nell'SRS (es. ALG02 BATCH-01 gestione tentativi, CDU-06 PDF) sono stati condivisi con CSI? Qual è la loro risposta?
5. Il documento `Mermaid.txt` contiene il diagramma architetturale. È la versione concordata con CSI o è una proposta?

---

## ADR correlati

| ADR | Decisione |
|---|---|
| [ADR-005](ADR-005-sicurezza-cdu-15-16.md) | Sicurezza CDU-15/16 (rischio MODERATO 3 mitigato) |
| [ADR-007](ADR-007-batch-01-5min-skip-locked.md) | BATCH-01 5min (rischio AGGIUNTO 4 — ambiguità SRV-01/SRV-03 da chiudere) |
| [ADR-013](ADR-013-migrazione-pg-dump-restore.md) | Migrazione PG (rischio CRITICO 1 — DMP non formalizzato) |
| [ADR-016](ADR-016-scaduto-async-batch-02.md) | SCADUTO async (rischio AGGIUNTO 5 — semantica AS-IS vs TO-BE) |
