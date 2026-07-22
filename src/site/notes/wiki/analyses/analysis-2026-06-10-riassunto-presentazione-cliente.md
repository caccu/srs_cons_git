---
{"dg-publish":true,"permalink":"/wiki/analyses/analysis-2026-06-10-riassunto-presentazione-cliente/","title":"Riassunto Esecutivo — Presentazione Progetto Gestione Consensi","tags":["presentazione","riassunto","executive-summary","gestione-consensi","cliente"],"dg-note-properties":{"title":"Riassunto Esecutivo — Presentazione Progetto Gestione Consensi","aliases":["Riassunto Esecutivo — Presentazione Progetto Gestione Consensi"],"type":"analysis","tags":["presentazione","riassunto","executive-summary","gestione-consensi","cliente"],"created":"2026-06-10","updated":"2026-07-16","sources":["2026-03-02-conspref-srs-v1-revised","2026-03-02-appunti-e-pianificazione","2026-03-02-domande-srs-csi-v02","2023-09-01-conspref-srs-01-v03"],"related":["[[overview|Overview]]","[[gestione-consensi-applicativo|Gestione Consensi - Applicativo]]","[[analysis-2026-05-14-punti-aperti-csi|Punti Aperti da Chiedere a CSI Piemonte — Tracker Unificato]]","[[valutazione-qualita-srs-consensi|Valutazione Qualità SRS — Gestione Consensi]]","[[wiki/analyses/analysis-2026-05-27-punti-aperti-spiegati\|Punti Aperti — Spiegati in Modo Semplice]]"]}}
---


# Riassunto Esecutivo — Progetto Gestione Consensi

**Documento di sintesi per presentazione al cliente** — preparato il 10/06/2026.
Sintetizza l'intera base di conoscenza del progetto: contesto, funzionalità, architettura, decisioni prese, pianificazione e punti ancora aperti.

> ✅ **Aggiornamenti successivi (non riflessi nel corpo di questo snapshot 10/06):** **CDU-17 rielaborato e confermato** dal committente (call 20/07/2026: Variante B eliminata, passo 5, endpoint CRUD via API Manager, scenario manutenzione `IN_MANUTENZIONE`; ADR-006 → `accepted`). Sicurezza API aggiornata a 07/2026: token via **API Manager APIMBBONE**, `codice_ente` inoltrato dal Gateway, **`cons_t_client_ente` fuori scope V1.0**. Fonti correnti: [[wiki/concepts/alternativa-batch-03-pull\|CDU-17 PULL]] · [[wiki/concepts/sicurezza-cdu-15-16\|Sicurezza CDU-15/16]] · [[CDU-17_diagramma-sequenza\|diagramma CDU-17]].

---

## 1. Di cosa si tratta

La Regione Piemonte dispone di un applicativo che raccoglie e gestisce i **consensi sanitari dei cittadini** (ad esempio: il consenso all'apertura del Fascicolo Sanitario, al Dossier Clinico aziendale, alla telemedicina). L'applicativo attuale risale al 2019 e va completamente rifatto.

- **Committente:** [[wiki/entities/regione-piemonte\|Regione Piemonte]], tramite [[wiki/entities/csi-piemonte\|CSI Piemonte]] (gestore tecnico dell'infrastruttura)
- **Fornitore:** [[wiki/entities/exprivia\|Exprivia S.p.A.]] — analisi, specifica e sviluppo del nuovo sistema
- **Fase attuale:** analisi completata, specifica dei requisiti (SRS) in revisione finale con CSI, pre-sviluppo
- **Documento centrale:** CONSPREF-SRS-V1.0 (bozza v2, rivista con i 69 commenti CSI della v3 — 30 risposte già consolidate)

---

## 2. Cosa fa il sistema

### I tre livelli di consenso

| Livello | Ambito | Esempio |
|---|---|---|
| **Nazionale** | Tutto il territorio | Fascicolo Sanitario Elettronico (FSE/INI) |
| **Regionale** | Regione Piemonte | Telemedicina, stratificazione, reti di patologia |
| **Aziendale** | Singola azienda sanitaria (ASR) | Dossier Clinico, referti online |

Regola chiave: il consenso regionale si esprime **una volta sola** e vale per tutte le aziende; quello aziendale si esprime **azienda per azienda**.

### Chi lo usa e come

Il consenso si può esprimere attraverso **tre canali**:

1. **Webapp Cittadino** — il cittadino accede con SPID/CIE e gestisce i propri consensi (o quelli di un suo delegante, tramite il pulsante "Deleghe")
2. **Webapp Operatore** — l'operatore sanitario o amministrativo opera per conto dell'assistito, accedendo tramite PUA con credenziali RUPAR/IRIDE
3. **LIS** — acquisizione presso il laboratorio (canale di front-office)

A questi si aggiungono il **Back Office** (configurazione dei tipi di consenso, delle informative e degli enti) e le **API per i sistemi informativi aziendali (SIA)** delle ASR, che interrogano lo stato dei consensi in modalità applicativa (machine-to-machine).

### Le funzionalità in numeri

- **16 casi d'uso (CDU)** — dall'autenticazione (CDU-01, diviso in 01a Operatore e 01b Cittadino) alla consultazione del cruscotto (CDU-02), al rilascio e modifica del consenso (CDU-03/04), al download PDF dell'informativa (CDU-06), fino alle funzioni operatore (CDU-07÷11), back office (CDU-12÷14) e API per i SIA (CDU-15/16, più il proposto CDU-17)
- **3 processi batch** per le elaborazioni asincrone (notifiche, scadenze, allineamenti)
- **25 tabelle** nel nuovo modello dati (erano 12 nel sistema attuale)
- **6 sistemi esterni integrati**

### Il ciclo di vita del consenso

Ogni consenso segue una macchina a stati semplice e tracciabile ([[wiki/concepts/ciclo-vita-consenso\|Ciclo di Vita del Consenso]]):

```
NON ESPRESSO → ATTIVO / NEGATO → SCADUTO / ANNULLATO
```

Principio fondamentale: **nessuna sovrascrittura**. Ogni cambiamento crea un nuovo record e il precedente viene storicizzato — la storia completa delle scelte del cittadino è sempre ricostruibile (decisione [[wiki/docs/adr/ADR-015-storicizzazione-immutabile\|ADR-015]]).

---

## 3. Architettura e tecnologie

```
Cittadino (SPID/CIE) → GASP Salute → Webapp Angular → Spring Boot → PostgreSQL 17
Operatore (RUPAR/IRIDE) → PUA → Webapp Angular → Spring Boot
SIA aziendali → API REST (OAuth2 + JWT) → Spring Boot
```

| Componente | Tecnologia | Note |
|---|---|---|
| Frontend | **Angular 19** + componenti QUASAR CSI | Form Renderer unico condiviso tra webapp Cittadino e Operatore |
| Backend | **Spring Boot 3.4.10+ / Java 17** | Client SOAP con Apache CXF per i sistemi legacy |
| Database | **PostgreSQL 17** su DBaaS Nivola | Esterno al namespace applicativo |
| Infrastruttura | **IaaS Nivola** (DEV/TEST/PROD) — provisioning CSI | Verbale 11/06/2026 — non ECaaS/Kubernetes come da SRS §3.5 |

Due scelte architetturali da evidenziare:

- **Sicurezza API — doppia esposizione** ([[wiki/concepts/sicurezza-cdu-15-16\|Sicurezza CDU-15-16]], [[wiki/docs/adr/ADR-004-no-api-gateway\|ADR-004]]): **AS-IS** (fruitori esistenti): Spring Security applicativa diretta, no API Gateway; **TO-BE** (nuovi fruitori esterni, es. Telemedicina): API Manager CSI Piemonte + token JWS (verbale 11/06/2026).
- **Form Renderer unico** ([[wiki/docs/adr/ADR-008-ssot-form-renderer\|ADR-008]]): la pagina di espressione del consenso è composta dinamicamente da un unico motore, usato sia dal cittadino sia dall'operatore — una sola fonte, zero divergenze tra i due canali.

### Sistemi esterni integrati

| Sistema | Ruolo | Protocollo |
|---|---|---|
| [[wiki/concepts/gasp-salute\|GASP Salute]] | Identity Provider per il cittadino (SPID/CIE) | **SAML2** (confermato verbale 11/06/2026) |
| AURA | Anagrafe regionale: ricerca assistito | SOAP |
| Gestione Deleghe | Elenco deleganti del cittadino | SOAP |
| [[wiki/entities/notificatore-unp\|Notificatore UNP]] | Notifiche al cittadino (email/push/app IO) | REST |
| SIA ASR | Sistemi informativi delle aziende sanitarie | REST (CDU-15/16/17) |
| PUA / Configuratore | Accesso e profilazione operatori | Web |

---

## 4. Sicurezza

- Autenticazione cittadino via **SPID/CIE** (GASP Salute); operatori via **RUPAR/IRIDE** (PUA)
- Per le API verso i SIA ([[wiki/concepts/sicurezza-cdu-15-16\|Sicurezza CDU-15-16]]): **OAuth2 Client Credentials + token JWT**, con isolamento per ente garantito da una **difesa a tre livelli** — validazione del token, tabella di mapping client→ente (`cons_t_client_ente`), filtro applicativo sulle query. Ogni azienda vede **solo i propri dati**, per costruzione.
- Errori delle API in formato standard **RFC 7807** ([[wiki/docs/adr/ADR-018-rfc-7807-error-response\|ADR-018]])
- OWASP Top 10, codice fiscale mascherato nei log, credenziali gestite lato infrastruttura IaaS CSI

---

## 5. Processi batch e la proposta CDU-17

| Batch | Funzione | Stato |
|---|---|---|
| **BATCH-01** | Notifica alle aziende dei consensi acquisiti — ogni 5 minuti, con SKIP LOCKED ([[wiki/docs/adr/ADR-007-batch-01-5min-skip-locked\|ADR-007]]) | Confermato |
| **BATCH-02** | Gestione scadenze: porta i consensi a SCADUTO in modo asincrono ([[wiki/docs/adr/ADR-016-scaduto-async-batch-02\|ADR-016]]) | Confermato (un punto da approfondire: SC67) |
| **BATCH-03** | Allineamento massivo push verso le aziende | **In sostituzione** |

**Proposta CDU-17 — modello PULL "centro stella"** ([[wiki/concepts/alternativa-batch-03-pull\|Alternativa BATCH-03]], [[wiki/docs/adr/ADR-006-batch-03-pull-cdu-17\|ADR-006]], stato: proposta in attesa di approvazione CSI): invece di spingere i dati verso ogni azienda (push), il sistema regionale espone un endpoint REST paginato e sono le aziende a **scaricare** lo snapshot quando serve. Vantaggi: nessun onere infrastrutturale push sul sistema regionale, zero downtime, riuso integrale della sicurezza già progettata per CDU-15/16.

---

## 6. Migrazione dati PostgreSQL 9 → 17

Il database attuale è su PostgreSQL 9: il salto è di **8 versioni major**. Strategia scelta: **dump & restore logico** ([[wiki/docs/adr/ADR-013-migrazione-pg-dump-restore\|ADR-013]], [[wiki/concepts/migrazione-postgres-9-17\|Migrazione PostgreSQL 9 → 17]]), con procedura di cutover completa e piano di rollback (istanza PG9 in standby per 48 ore).

Rischi tecnici già censiti: cambio autenticazione (md5→scram), tipi deprecati (SERIAL), comportamento timestamp, sequenze orfane. Punto aperto: il **piano di migrazione formale (CONSPREF-DMP)** non è ancora stato formalizzato — redazione in carico a **CSI Piemonte** (confermato 16/07/2026).

---

## 7. Pianificazione

**8 fasi, 120 giorni lavorativi (~17 settimane), 10 sprint da 2 settimane:**

| Fase | Contenuto | Settimane |
|---|---|---|
| 0 | Setup e prerequisiti (provisioning, accessi) | 1–2 |
| 1 | Design tecnico | 2–3 |
| 2 | Data layer TO-BE | 3–5 |
| 3 | Backend Spring Boot | 4–11 |
| 4 | Frontend Angular (in parallelo al backend) | 5–12 |
| 5 | Testing & QA | 11–15 |
| 6 | Migrazione PG9→PG17 | 13–16 |
| 7 | Go-Live | 15–17 |

Attenzione alle **latenze esterne**: provisioning DBaaS Nivola, credenziali IRIS/AURA e documentazione GASP vanno richiesti **dal giorno 1** perché non dipendono da Exprivia.

---

## 8. Decisioni architetturali registrate (ADR)

Il progetto mantiene un registro formale di **19 decisioni architetturali** (18 approvate + 1 proposta). Le principali:

| Decisione | Sintesi |
|---|---|
| Stack tecnologico | Spring Boot 3 + Java 17 + Angular 19 + PostgreSQL 17 |
| Sicurezza API | AS-IS: Spring Security diretta (no API Gateway); TO-BE: API Manager CSI per nuovi fruitori (verbale 11/06/2026) |
| Sicurezza API SIA | OAuth2 Client Credentials + JWT + isolamento per ente |
| **CDU-17 PULL** | Sostituzione BATCH-03 push — **unica decisione ancora in stato "proposta"** |
| Storicizzazione immutabile | Nessuna sovrascrittura dei consensi |
| Split CDU-01 | Accesso Operatore (01a) e Cittadino (01b) separati |
| Form Renderer SSoT | Motore unico di composizione form Cittadino + Operatore |
| Migrazione dump/restore | PG9 → PG17 senza pg_upgrade diretto |

Elenco completo: [[wiki/docs/adr/README\|Indice ADR]]

---

## 9. Punti aperti

Il tracker unificato ([[wiki/analyses/analysis-2026-05-14-punti-aperti-csi\|Punti Aperti da Chiedere a CSI Piemonte]]) censisce **40 punti aperti** (~36 al netto dei duplicati), raggruppati per area e priorità. I **più critici** — quelli che bloccano la partenza:

| Punto | Perché è critico |
|---|---|
| ✅ ~~GASP Salute: OIDC o SAML2?~~ | **CHIUSO** — SAML2 confermato (verbale 11/06/2026). Richiedere metadata XML IdP in Sprint 0. |
| **Provisioning DBaaS Nivola (DEV e PROD)** | Senza database non parte lo Sprint 1 |
| ✅ ~~Accesso automation CSI~~ | **CHIUSO** — Skeleton in carico a Exprivia (IaaS; verbale 11/06/2026) |
| ✅ ~~CONSPREF-DMP senza responsabile~~ | **CHIUSO** — redazione in carico a CSI Piemonte (16/07/2026); resta la bozza v1 (rischio slittamento Fase 6 finché non prodotta) |
| **URL Authorization Server + algoritmo firma JWT** | Bloccano qualsiasi test di integrazione con i SIA |
| **CDU-17: conferma modello PULL** | I SIA sanno fare chiamate REST attive? (PULL-08) Va scritta la specifica end-to-end (PULL-09) |
| **OpenAPI CDU-15/16: 5 TBD** | Da confermare prima di condividere la specifica con le ASR |
| **BATCH-01: ambiguità WSDL (SRV-01 vs SRV-03)** | Rischio di errore implementativo sull'integrazione |

Versione divulgativa punto per punto: [[wiki/analyses/analysis-2026-05-27-punti-aperti-spiegati\|Punti Aperti — Spiegati in Modo Semplice]].

---

## 10. Stato della base di conoscenza

Questa wiki raccoglie e mantiene allineata tutta la conoscenza di progetto:

- **12 documenti sorgente** ingestionati e sintetizzati (SRS TO-BE, requisiti CSI 2023, baseline AS-IS 2019, linee guida cloud CSI, specifiche WSDL, dizionario dati)
- **12 pagine concettuali** (architettura, ciclo di vita, sicurezza, batch, migrazione, …)
- **9 analisi** (valutazione qualità SRS, gap AS-IS→TO-BE, tracker punti aperti, specifica OpenAPI CDU-15/16 in bozza)
- **19 ADR** con ciclo di vita gestito
- Audit di coerenza completo contro l'SRS v3 (29/05/2026): zero disallineamenti residui
- Analisi strutturale a grafo (08/06/2026): da qui sono emersi i due punti critici PULL-08 e PULL-09 sul CDU-17

**Giudizio complessivo sull'analisi** ([[wiki/analyses/valutazione-qualita-srs-consensi\|Valutazione Qualità SRS]]): documento di alta qualità per una bozza — 7 punti di forza, 5 rischi censiti e tracciati, checklist operativa pre-sprint pronta.
