---
{"dg-publish":true,"permalink":"/wiki/analyses/analysis-2026-05-14-punti-aperti-csi/","title":"Punti Aperti da Chiedere a CSI Piemonte — Tracker Unificato","tags":["tracker","punti-aperti","csi-piemonte","blocking","sprint-0","sprint-1","da-chiedere"],"dg-note-properties":{"title":"Punti Aperti da Chiedere a CSI Piemonte — Tracker Unificato","aliases":["Punti Aperti da Chiedere a CSI Piemonte — Tracker Unificato"],"type":"analysis","tags":["tracker","punti-aperti","csi-piemonte","blocking","sprint-0","sprint-1","da-chiedere"],"created":"2026-05-14","updated":"2026-07-20","sources":["2026-03-02-conspref-srs-v1-revised","2026-03-02-domande-srs-csi-v02"],"related":["[[wiki/analyses/analysis-2026-05-06-checklist-avvio-progetto\|Checklist Avvio Progetto — Gestione Consensi]]","[[wiki/concepts/sicurezza-cdu-15-16\|Sicurezza CDU-15-16 — Modello Autorizzazione per Ente]]","[[alternativa-batch-03-pull|Alternativa BATCH-03 — PULL CDU-17 (centro stella)]]","[[wiki/analyses/analysis-2026-05-06-openapi-cdu-15-16\|analysis-2026-05-06-openapi-cdu-15-16]]","[[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|analysis-2026-05-14-risposte-mf-srs-v3]]","[[GASP Salute]]","[[wiki/concepts/batch-processes\|Processi Batch — BATCH-01, BATCH-02, BATCH-03]]","[[Sistemi Esterni Integrati]]","[[wiki/sources/2026-05-05-mermaid-architettura\|Diagramma Architettura Sistema — Mermaid]]","[[valutazione-qualita-srs-consensi|Valutazione Qualità SRS — Gestione Consensi]]"]}}
---


# Punti Aperti da Chiedere a CSI Piemonte

**Scopo:** consolidare in un unico tracker tutte le domande, ambiguità, TBD e proposte in attesa di conferma sparse nel corpus wiki. Ogni voce rimanda alla pagina autoritativa.

**Stato corpus alla data:** 31 pagine, 30/30 risposte MF da SRS v3_lavorazione propagate, 0 dead-link.

> 🔄 **Aggiornamento 18/06/2026 (allineato all'agenda riunione):** chiusi/recepiti nel documento — **ID-01** (GASP=SAML2, restano solo metadata), **INF-03** (skeleton Exprivia/IaaS), **INT-05** (Notificatore di Deleghe ≠ UNP), **INF-04** (diagramma aggiornato). Nuovi/raffinati — **INF-05** (dettagli operativi IaaS: deploy/ingress/segreti/CI-CD + pila «k8s»), **BAT-01** (SRV-03 + SRV-04), **BAT-02/SC67** (sorgente `annulla_consensi`: informativa scaduta vs nuova), **GOV-02** (deroga V03 su `online`/`annulla_consensi`). L'ordine del giorno operativo è in `Agenda-riunione-CSI-CONSPREF_2026-06-18`.

> ✅ **Aggiornamento call CSI 20/07/2026 — chiusure di massa.** Chiusi/delegati: **SEC-01÷06** (Q1 header CF+`codice_ente` risolto; Q3/Q4/Q5/Q6 delegati ad APIMBBONE), **INT-01** (AURA: nessun servizio nuovo, stessi WSDL nei properties, credenziali IRIS DEV incluse → chiude anche **ID-03**), **INT-02** (Gestione Deleghe già integrata), **GOV-02** (deroga V03 `online`/`annulla_consensi` approvata, sorgente unica `cons_d_informativa` confermata), **BAT-01** (SRV-03/SRV-04 confermati, "svecchiare + nuova logica"). **Differiti/non vincolanti:** onboarding SIA (evento raro → CR), Manutenzione ASR §7.4 (post-Sprint 0), frequenza BATCH-02, **GOV-05/API-05** (lista ASR), Stato SCADUTO **BAT-03** (call dedicata pending). **Da riformulare:** **BAT-02/SC67** (CSI non ha capito la domanda → domanda mirata a scenario, vedi sotto). **Non toccato:** CDU-17 (attende delucidazioni via mail).

**Legenda priorità:**
- 🔴 **CRITICO** — blocca Sprint 0/1, da chiarire **Giorno 1**
- 🟠 **ALTO** — blocca Sprint 2-3
- 🟡 **MODERATO** — necessario prima di UAT/go-live
- ⚪ **APERTO** — utile ma non bloccante

---

## 1. Identità & Autenticazione (Cittadino + Operatore)

| #     | Domanda                                                                                | Prio | Sprint   | Fonte wiki                                                                         |
| ----- | -------------------------------------------------------------------------------------- | ---- | -------- | ---------------------------------------------------------------------------------- |
| ID-01 | ~~GASP Salute: protocollo OIDC o SAML2? metadata?~~ ✅ **CHIUSO** — **SAML2** confermato (verbale 11/06/2026); **metadata SP ricevuti** (TEST/preprod, 07/2026: Shibboleth SP `tst-consprefbo-spid.isan.csi.it`, IdP GASPRP_SALUTE, LIV1/2/3). Resta: censire il SP via `Template-richiesta-Federazione-Service-Provider` a `identita.federazione@csi.it`. | ✅   | Giorno 1 | [[wiki/concepts/gasp-salute\|GASP Salute]], [[wiki/analyses/analysis-2026-05-06-checklist-avvio-progetto\|analysis-2026-05-06-checklist-avvio-progetto]] §B1 |
| ID-02 | Registrazione app PUA — 2 profili (Operatore, Amministratore)                          | 🟠   | Sprint 2 | Checklist §B9                                                                      |
| ID-03 | ~~Credenziali IRIS per autenticazione AURA (ambiente DEV)~~ ✅ **CHIUSO (call 20/07/2026):** incluse tra i parametri già presenti nei file di properties AURA | ✅   | Sprint 1 | Checklist §B7                                                                      |

---

## 2. Sicurezza CDU-15/16 — OAuth2/JWT (TR30 → TR58)

Tutte da [[wiki/concepts/sicurezza-cdu-15-16\|Sicurezza CDU-15-16 — Modello Autorizzazione per Ente]] §Punti da chiarire con CSI.

> ✅ **CHIUSI in call CSI 20/07/2026.** Il token è gestito dall'**API Manager CSI APIMBBONE** (OAuth2 `client_credentials`, Key Manager + Gateway). Q1 (header/claim) risolto; Q3/Q4/Q5/Q6 delegati ad APIMBBONE. **Unico residuo attivo:** produrre e consegnare lo **swagger (OpenAPI)** dei servizi CDU-15/16/17 per abilitare la sottoscrizione. Dettaglio in [[wiki/concepts/sicurezza-cdu-15-16\|sicurezza-cdu-15-16]] §7.

| #     | Domanda                                                                                                    | Esito |
|-------|------------------------------------------------------------------------------------------------------------|------|
| SEC-01 | ~~URL Authorization Server / header-claim~~ | ✅ **RISOLTO** — il Gateway APIM inoltra sempre **CF (da Shibboleth) + `codice_ente`**; backend deriva l'ente da qui |
| SEC-02 | ~~Firma JWT + JWKS~~ | ✅ **Non a nostro carico** — token validato dal Key Manager APIM. Prerequisito: **swagger** |
| SEC-03 | ~~Onboarding nuovo SIA + chi popola `cons_t_client_ente`~~ | ✅ **Delegato ad APIMBBONE** (http interno); mapping fuori scope V1.0; evento **raro** → **CR** |
| SEC-04 | ~~TTL token + refresh~~ | ✅ **Delegato ad APIMBBONE** (policy interne) |
| SEC-05 | ~~Scope OAuth~~ | ✅ **Delegato ad APIMBBONE** |
| SEC-06 | ~~Revoca credenziali compromesse~~ | ✅ **Delegato** — credenziali fornite da CSI, revoca a **terza parte** |

---

## 3. CDU-17 Snapshot Pull (TR34 → TR68) — sostituzione BATCH-03

> ✅ **CDU-17 RIELABORATO E CONFERMATO dal committente (call CSI 20/07/2026).** Fonte autoritativa: [[wiki/concepts/alternativa-batch-03-pull\|Alternativa BATCH-03 — PULL CDU-17]] + diagrammi [[CDU-17_diagramma-sequenza\|CDU-17]] · [[Manutenzione-endpoint_diagramma-sequenza\|Manutenzione endpoint]]. La maggior parte dei punti è chiusa.

| #     | Domanda                                                                                                    | Esito (call 20/07/2026) |
|-------|------------------------------------------------------------------------------------------------------------|------|
| PULL-01 | ~~Variante blocco o watermark?~~ | ✅ **Blocco obbligatorio** (Variante A). Watermark (B) **eliminata** |
| PULL-02 | ~~Canale notifica out-of-band~~ | ✅ **email e/o webhook** via parametro di configurazione. Con webhook **il SIA espone un REST** (contratto/firma/sicurezza forniti da CSI) |
| PULL-03 | Scope OAuth `consensi:snapshot` + lifecycle | Delegato ad **APIMBBONE**; resta lo swagger |
| PULL-04 | `page_size` massimo + tarabilità | ⚪ Da precisare (non bloccante) |
| PULL-05 | ~~Variante export-with-downtime~~ | ✅ **Non perseguita** |
| PULL-06 | ~~BATCH-03 eliminazione o marker?~~ | ✅ **Rimosso** dal SRS (§6.17) |
| PULL-07 | Conferma via PATCH idempotente | ✅ Confermato idempotente |
| PULL-08 | ~~SIA può fare chiamate REST attive?~~ | ✅ **Sì** — SIA/aziende via API Manager |
| PULL-09 | **Swagger (OpenAPI) CDU-17** da scrivere end-to-end. Ora include: **passo 5** (notifica esito webapp), **servizi endpoint CRUD** esposti alle aziende, **scenario manutenzione** (`IN_MANUTENZIONE`). | 🟠 **Residuo attivo** — Sprint 0/1 |

---

## 4. OpenAPI CDU-15/16 — TBD da OpenAPI v0.1

Da [[wiki/analyses/analysis-2026-05-06-openapi-cdu-15-16\|analysis-2026-05-06-openapi-cdu-15-16]] e dal file `openapi-cdu-15-16-v0.1.yaml`.

| #     | Domanda                                                                                                    | Prio | Sprint   |
|-------|------------------------------------------------------------------------------------------------------------|------|----------|
| API-01 | URL Authorization Server CSI (TODO-M1) — confluisce con SEC-01                                            | 🔴   | Sprint 0 |
| API-02 | Scope OAuth richiesto per `/consensi/stato` (TODO-M2) — confluisce con SEC-05                             | 🟠   | Sprint 1 |
| API-03 | Schema paginazione cursor-based — `page_size` default + max accettato (TODO-M3)                          | 🟡   | Sprint 2 |
| API-04 | SLA tempo risposta + throughput target per CDU-15/16 (TODO-M4)                                            | 🟡   | UAT      |
| API-05 | Lista ASR coinvolte nel TO-BE + referenti tecnici (TODO-M5). **Call 20/07/2026:** non vincolante, differito. | ⚪ (differito)   | Sprint 2 |

---

## 5. Batch & WSDL — BATCH-01 / BATCH-02

| #     | Domanda                                                                                                    | Prio | Sprint   | Fonte                                                              |
|-------|------------------------------------------------------------------------------------------------------------|------|----------|--------------------------------------------------------------------|
| BAT-01 | ~~Confermare SRV-03 NotificaAcquisizioneConsenso + SRV-04 NotificaRevocaConsenso~~ ✅ **CONFERMATO (call 20/07/2026):** operazioni esistenti, **"solo da svecchiare e integrare la nuova logica"**. Restano solo i nomi esatti dei campi del tracciato. | ✅ | Sprint 1 | [[wiki/concepts/batch-processes\|Processi Batch — BATCH-01, BATCH-02, BATCH-03]] §RISCHIO, Checklist §B10 |
| BAT-02 | ~~**SC67**~~ ✅ **RISOLTO PER VIA TECNICA (2026-07-23).** La storicizzazione BATCH-02 è **ancorata all'informativa scaduta A**: legge `annulla_consensi` da A e àncora il record terminale a A. Allinea l'SQL §7.2 alla prosa autoritativa §6.13; nessun impatto su CDU-17 (snapshot esporta solo consensi attivi). Da segnalare a CSI come scelta di design, non domanda. **SQL SRS §7.2 corretto (2026-07-23, `.md`+`.docx`, riga variazioni 1.4).** | ✅ Chiuso | Prima di chiudere SRS | [[wiki/concepts/batch-processes\|Processi Batch — BATCH-01, BATCH-02, BATCH-03]] §ALG02, [[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|analysis-2026-05-14-risposte-mf-srs-v3]] §Tema J |
| BAT-03 | Stato SCADUTO — semantica cambiata AS-IS vs TO-BE. **Call 20/07/2026:** CSI lo considera "componente da gestire" → **call dedicata pending** per definire la comunicazione ai SIA ASR (gestione asincrona). | 🟠   | Sprint 1 (call pending) | [[wiki/concepts/batch-processes\|Processi Batch — BATCH-01, BATCH-02, BATCH-03]] §Differenza semantica, [[wiki/analyses/analysis-gap-as-is-to-be\|Analisi Gap AS-IS → TO-BE — Gestione Consensi]] |

---

## 6. Integrazioni & Canali

| #      | Domanda                                                                                   | Prio | Sprint   | Fonte                                                                                                             |
| ------ | ----------------------------------------------------------------------------------------- | ---- | -------- | ----------------------------------------------------------------------------------------------------------------- |
| INT-01 | ~~**WSDL AURA** — lista servizi (CDU-07/08)~~ ✅ **CHIUSO (call 20/07/2026):** nessun servizio nuovo, **gli stessi già presenti nei file di properties** (`FindProfiliAnagrafici`, `getProfiloSanitario`); credenziali IRIS DEV incluse. | ✅   | Sprint 0 | Checklist §B5                                                                                                     |
| INT-02 | ~~**WSDL Gestione Deleghe** — `getDelegantiService`, accreditamento portale API-Piemonte~~ ✅ **CHIUSO (call 20/07/2026):** **già integrato, nulla da fare.** | ✅   | Sprint 0 | Checklist §B6, [[wiki/concepts/sistemi-esterni-integrati\|Sistemi Esterni Integrati]] §Gestione Deleghe |
| INT-03 | **LIS — acronimo + spec integrazione canale acquisizione consensi** (MF3R1, MF4R1)        | 🟠   | Sprint 1 | [[wiki/concepts/sistemi-esterni-integrati\|Sistemi Esterni Integrati]] §LIS, [[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|analysis-2026-05-14-risposte-mf-srs-v3]] §Tema A |
| INT-04 | Accesso repo **QUASAR CSI** (componenti UI)                                               | 🟠   | Sprint 1 | Checklist §B8                                                                                                     |
| INT-05 | ~~Distinzione Notificatore di Deleghe ≠ Notificatore UNP in SRS~~ ✅ **RECEPITO** in SRS §4.2/§7 (Deleghe = conferma rilascio post-COMPLETATO; UNP = annullamento/scadenza e notifiche generiche) | ✅   | — | [[wiki/concepts/sistemi-esterni-integrati\|Sistemi Esterni Integrati]] §Notificatore                                            |
|        |                                                                                           |      |          |                                                                                                                   |

---

## 7. Infrastruttura & Provisioning

| #     | Domanda                                                                                                    | Prio | Sprint   | Fonte         |
|-------|------------------------------------------------------------------------------------------------------------|------|----------|---------------|
| INF-01 | **Provisioning DBaaS Nivola DEV** — ✅ **in corso (07/2026)**; su DEV verrà caricato un **ribaltamento dei dati** dal DB **TEST PG 9.6**. | 🟠 | Giorno 1 | Checklist §B2 |
| INF-02 | **Provisioning DBaaS Nivola PROD** — **rinviato**: in questa fase si creano solo **DEV + pre-prod** (no PROD, per contenere i costi). | 🟡 | Post-collaudo | Checklist §B3 |
| INF-03 | ~~Accesso automation CSI~~ ✅ **CHIUSO** — Skeleton in carico a **Exprivia** (IaaS, non ECaaS). Confronto su POM con CSI (verbale 11/06/2026). | ✅   | Giorno 1 | Checklist §B4 |
| INF-04 | ~~Diagramma architetturale~~ ✅ **AGGIORNATO** (Exprivia, 18/06/2026): rimosso il nodo API Gateway dal percorso AS-IS, infra IaaS, SIA 1:n, cittadini via SPID/CIE diretto, aggiunti EnteAuthorizationFilter/Snapshot Service/CDU-17. | ✅ | — | [[wiki/sources/2026-05-05-mermaid-architettura\|Diagramma Architettura Sistema — Mermaid]], Checklist §B14 |
| INF-05 | Dettagli operativi ambiente IaaS — ✅ **in gran parte chiariti (07/2026, `ElencoUrlTools`)**: deploy via **automation Chef (ADA Deployer)** su Nivola; CI/CD **GitLab + Jenkins**; qualità **SonarQube**; artefatti **Artifactory**; consegna **ASGARD**. **Restano:** ingress/TLS e meccanismo di gestione dei **segreti applicativi**. | 🟠 | Sprint 0/1 | [[wiki/concepts/architettura-iaas\|Architettura IaaS]] §Toolchain, SRS §3.5 |

---

## 8. Governance & Documenti

| #      | Domanda                                                                                                                   | Prio | Sprint         | Fonte                                                                        |
| ------ | ------------------------------------------------------------------------------------------------------------------------- | ---- | -------------- | ---------------------------------------------------------------------------- |
| GOV-01 | **Approvazione formale SRS V1.0 bozza v2** da CSI (post-recepimento risposte MF v3_lavorazione)                           | 🟠   | Prima Sprint 1 | Checklist §B12, [[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|analysis-2026-05-14-risposte-mf-srs-v3]]                   |
| GOV-02 | **Validazione [PROPOSTA] nell'SRS** — ALG02 BATCH-01, CDU-06 PDF, 11 proposte §8.4. ✅ **Deroga V03 su `online`/`annulla_consensi` APPROVATA (call 20/07/2026):** confermata la scelta V1.0 di mantenerli su `cons_d_informativa` (sorgente autoritativa unica); CSI conferma il funzionamento. Restano le altre [PROPOSTA]. | 🟠 (deroga V03 ✅) | Prima Sprint 2 | Checklist §B13, [[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|analysis-2026-05-14-risposte-mf-srs-v3]] §Tema D           |
| GOV-03 | **CONSPREF-DMP** — ~~chi è responsabile lato CSI?~~ ✅ **CHIUSO 16/07/2026:** redazione in carico a **CSI Piemonte**. Resta la produzione della bozza v1 (Sprint 0) | ✅   | Sprint 0       | [[wiki/sources/2026-03-02-domande-srs-csi-v02\|2026-03-02-domande-srs-csi-v02]], Q11,[[wiki/analyses/valutazione-qualita-srs-consensi\|valutazione-qualita-srs-consensi]] |
| GOV-04 | **SLA e NFR performance** — tempo risposta max CDU-02, throughput BATCH-01, disponibilità (99.x%)                         | 🟡   | Prima UAT      | Checklist §B15                                                               |
| GOV-05 | **Lista ASR coinvolte** + referenti tecnici (confluisce con API-05). **Call 20/07/2026:** componente **non vincolante**, smarcabile in seguito. | ⚪ (differito)   | Sprint 2       | Checklist §B11                                                               |

---

## Riepilogo per Sprint

| Sprint        | Numero punti aperti                                                                                                                          |
|---------------|----------------------------------------------------------------------------------------------------------------------------------------------|
| Giorno 1 / Sprint 0 critico | ID-01, INF-01, INF-02, INF-03, GOV-03, SEC-01, SEC-02, PULL-01, PULL-02, PULL-08, PULL-09, API-01 → **12 punti** |
| Sprint 0 alto             | INT-01, INT-02, INF-04 → **3 punti**                                                                                                            |
| Sprint 1                  | ID-03, SEC-03÷05, PULL-03, PULL-06, API-02, BAT-03, INT-03, INT-04, GOV-01 → **11 punti**                                                       |
| Sprint 2+                 | ID-02, SEC-06, PULL-04, PULL-05, PULL-07, API-03, API-05, BAT-01, BAT-02, INT-05, GOV-02, GOV-05 → **12 punti**                                |
| Prima UAT                 | API-04, GOV-04 → **2 punti**                                                                                                                  |
| **Totale**                | **40 punti aperti** (dedupli: PULL-03↔SEC-05, API-01↔SEC-01, API-02↔SEC-05, API-05↔GOV-05 → ~36 distinti)                                       |

> ✅ **Stato residuo post-call CSI 20/07/2026.** Chiusi/delegati in questa call: **SEC-01÷06, INT-01, INT-02, ID-03, BAT-01, GOV-02** (deroga V03). Differiti non vincolanti: **GOV-05/API-05** (lista ASR), Manutenzione ASR §7.4, frequenza BATCH-02, onboarding SIA (→ CR). **Ancora aperti e attivi:**
> - **Swagger (OpenAPI) CDU-15/16/17** — prerequisito APIM (bloccante per la sottoscrizione).
> - ~~**BAT-02/SC67**~~ ✅ **risolto per via tecnica (2026-07-23):** storicizzazione ancorata all'informativa scaduta A; SQL SRS §7.2 corretto (`.md`+`.docx`).
> - **BAT-03 (Stato SCADUTO)** — call dedicata pending.
> - **CDU-17 / PULL-01÷09** — in attesa di delucidazioni via mail; lavorazione successiva.
> - **INF-01/INF-02** (DBaaS DEV in corso, PROD rinviato), **INF-05** (ingress/TLS + segreti), **GOV-01** (approvazione formale SRS v6).

---

## Note di gestione

1. **Ownership tracker:** Marco Forneris (Exprivia) propone questo file come agenda per la prossima riunione CSI/Exprivia.
   - 📋 **2026-06-18:** da questo tracker è stata derivata un'agenda formale per la riunione — deliverable `Agenda-riunione-CSI-CONSPREF_2026-06-18.docx`/`.pdf` (root repo), con in più i punti emersi dall'audit del 18/06: dettagli operativi infrastruttura **IaaS** (modello deploy/ingress/segreti/CI-CD + pila CSI di riferimento al posto degli identificativi «k8s») e l'evidenza su **BAT-01** (operazione WSDL attesa SRV-03 NotificaAcquisizioneConsenso).
2. **Workflow proposto:** ogni voce avrà un campo `risposta_csi` da popolare in revisione successiva del SRS; alla chiusura del punto, la voce si trasforma in entry permanente in [[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|analysis-2026-05-14-risposte-mf-srs-v3]] o nella relativa concept page.
3. **Dipendenze incrociate evidenti:**
   - SEC-01/API-01 (URL AS) → bloccante per qualsiasi test integrazione SIA
   - SEC-05/PULL-03/API-02 (scope) → da chiarire in singolo round
   - INT-01/INT-02 (WSDL AURA/Deleghe) → bloccano CDU-07/08
   - INF-01/INF-03 (DBaaS + automation) → bloccano partenza Sprint 1 tout court
   - PULL-08 (SIA caller capability) → bloccante per PULL-09 (spec CDU-17) e per estensione `EnteAuthorizationFilter` su chiamata inbound
   - PULL-09 (spec CDU-17) → bloccante per implementazione endpoint, per aggiornamento YAML OpenAPI e per test integrazione SIA
1. **Riferimenti operativi:** [[wiki/analyses/analysis-2026-05-06-checklist-avvio-progetto\|Checklist Avvio Progetto — Gestione Consensi]] resta lo strumento day-1 operativo; questo tracker è la versione consolidata e tematicamente raggruppata.
