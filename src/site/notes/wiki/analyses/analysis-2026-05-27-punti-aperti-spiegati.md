---
{"dg-publish":true,"permalink":"/wiki/analyses/analysis-2026-05-27-punti-aperti-spiegati/","title":"Punti Aperti — Spiegati in Modo Semplice","tags":["punti-aperti","csi-piemonte","guida-semplice","sprint-0","da-chiedere"],"dg-note-properties":{"title":"Punti Aperti — Spiegati in Modo Semplice","type":"analysis","tags":["punti-aperti","csi-piemonte","guida-semplice","sprint-0","da-chiedere"],"created":"2026-05-27","updated":"2026-07-20","sources":["2026-03-02-conspref-srs-v1-revised","2026-03-02-domande-srs-csi-v02"],"related":["[[analysis-2026-05-14-punti-aperti-csi|Punti Aperti da Chiedere a CSI Piemonte — Tracker Unificato]]","[[analysis-2026-05-06-checklist-avvio-progetto|Checklist Avvio Progetto — Gestione Consensi]]","[[gasp-salute|GASP Salute]]","[[batch-processes|Processi Batch — BATCH-01, BATCH-02, BATCH-03]]","[[sicurezza-cdu-15-16|Sicurezza CDU-15-16 — Modello Autorizzazione per Ente]]"]}}
---


# Punti Aperti — Spiegati in Modo Semplice

Versione "in parole povere" del [[wiki/analyses/analysis-2026-05-14-punti-aperti-csi\|tracker tecnico]], **allineata all'agenda della riunione CSI** (`Agenda-riunione-CSI-CONSPREF_2026-06-18`). Per ogni punto ancora aperto spiega: **cosa significa**, **perché serve**, **come si chiude**. È raggruppata come l'agenda.

**Priorità:** 🔴 critico (blocca la partenza) · 🟠 alto (blocca sprint 2-3) · 🟡 moderato (prima del collaudo) · ✅ chiuso/recepito.

---

## ✅ Già chiusi o recepiti (non più da chiedere)

> **Call CSI 20/07/2026 — nuove chiusure (spiegate sotto ai rispettivi punti):**
> - **Token e sicurezza API (Q1–Q6):** tutto in mano all'**API Manager (APIMBBONE)**. Ci dice sempre **chi sta chiamando** (Codice Fiscale preso da Shibboleth + codice dell'ente); durata token, scope e revoca delle credenziali li gestisce lui. A noi resta solo produrre lo **swagger**.
> - **AURA:** nessun servizio nuovo, sono **gli stessi già nei file di properties** (comprese le credenziali IRIS di DEV).
> - **Gestione Deleghe:** **già integrata**, non c'è nulla da fare.
> - **BATCH-01 (SRV-03/SRV-04):** operazioni confermate, **"solo da svecchiare"** con la nuova logica.
> - **online / annulla_consensi (deroga V03):** CSI **approva** la scelta V1.0 (parametri sull'informativa).

- **Login cittadini/operatori (GASP Salute):** ✅ protocollo **SAML2** confermato + **metadata ricevuti** (07/2026, ambiente TEST/preprod). L'integrazione è tramite un **Shibboleth SP** del CSI (non codice SAML nostro). Resta solo da censire il SP con il modulo di federazione.
- **Chi crea lo scheletro del progetto:** lo fa **Exprivia** (ambiente IaaS, non ECaaS), con confronto sul POM con CSI.
- **Due notificatori distinti:** nel documento ora è chiaro — *Notificatore di Deleghe* per la conferma di rilascio, *UNP* per annullamenti e notifiche generiche.
- **Diagramma dell'architettura:** aggiornato (tolto l'API Gateway dal percorso interno, infrastruttura IaaS, SIA 1‑a‑n, aggiunti Snapshot Service/CDU‑17).
- **BATCH-03:** rimosso e sostituito dal CDU-17 (snapshot PULL) nel documento.

---

## 1. Decisioni da far prendere a CSI

### 🔴 Sì al CDU-17 al posto di BATCH-03
**Cosa significa:** abbiamo proposto di sostituire il vecchio invio massivo (BATCH-03) con un servizio che i sistemi esterni vengono a interrogare loro ("centro stella"). **Come si chiude:** CSI dà l'ok formale (sign-off) alla proposta.

### 🟠 Approvazione del documento SRS (GOV-01)
**Cosa significa:** dopo aver recepito le risposte, l'SRS va approvato ufficialmente. **Come si chiude:** CSI approva la versione allineata del 18/06/2026.

### ✅ Ok alla scelta su online / annulla_consensi (GOV-02) — APPROVATO
**Cosa significa:** nella V1.0 teniamo questi due parametri sull'informativa; il requisito originale (V03) diceva di legarli al consenso. È una deroga consapevole. **Stato (call 20/07/2026):** CSI **approva** la scelta V1.0 e conferma il funzionamento — `cons_d_informativa` resta la sorgente autoritativa unica.

---

## 2. Bloccanti — da chiarire subito (Sprint 0)

### ✅ Metadata di GASP Salute (ID-01) — RICEVUTI
**Cosa significa:** protocollo SAML2; per integrarlo servivano i metadata del Service Provider e gli indirizzi. **Stato:** ✅ CSI ha fornito i metadata (ambiente TEST/preprod, 07/2026). L'autenticazione passa da un **Shibboleth SP** del CSI (host `tst-consprefbo-spid.isan.csi.it`, IdP GASPRP_SALUTE, livelli LIV1/2/3). **Resta:** compilare il modulo di federazione del SP e inviarlo a CSI.

### ✅ Sicurezza delle API (SEC-01÷06 / Q1–Q6) — CHIUSO (call 20/07/2026)
**Cosa significa:** i sistemi esterni che chiamano le nostre API ottengono prima un token. **Stato:** tutto in mano all'**API Manager del CSI (APIMBBONE)**, OAuth2 (client_credentials); le nostre API stanno **dietro il gateway**.
- **Chi sta chiamando (Q1):** ✅ il gateway ci **inoltra sempre** il **Codice Fiscale** (preso da Shibboleth) e il **codice dell'ente**. Da lì capiamo di quale ente sono i dati — non ci serve più una tabella di mappatura nostra.
- **Onboarding nuovi sistemi (Q3):** ✅ lo gestisce l'API Manager (esposto internamente in http); è un **evento raro**, eventualmente lo aggiungiamo dopo con una **CR**.
- **Durata e rinnovo del token (Q4):** ✅ **politiche interne** dell'API Manager.
- **Scope / permessi (Q5):** ✅ delegati all'API Manager.
- **Revoca credenziali rubate (Q6):** ✅ le credenziali le fornisce CSI; la revoca è affidata a un **servizio di terza parte**.

**Resta da fare da parte nostra:** solo produrre lo **swagger (OpenAPI)** dei servizi CDU-15/16/17, per poter sottoscrivere l'API sull'API Manager.

### 🟠 Database DEV e pre-prod (INF-01 / INF-02) — in corso
**Cosa significa:** il database lo fornisce CSI su Nivola. **Stato (07/2026):** provisioning **in corso**; per contenere i costi si creano **solo DEV e pre-prod** (la produzione più avanti). Sul DB di **DEV** ci verrà caricato un **ribaltamento dei dati** oggi presenti sul DB di **TEST (PostgreSQL 9.6)** — utile per lavorare con dati reali e per provare la migrazione a PostgreSQL 17. **Resta:** completamento del provisioning.

### 🟠 Dettagli dell'ambiente IaaS (INF-05) — in gran parte chiariti
**Cosa significa:** l'ambiente è IaaS (non ECaaS/Kubernetes). **Stato (07/2026, doc `ElencoUrlTools`):** il modello operativo è ora chiaro — il codice sta su **GitLab**, build e controlli qualità con **Jenkins + SonarQube**, artefatti su **Artifactory**, e il **rilascio avviene con automation Chef tramite ADA** (non Helm/GitOps/Kubernetes); la consegna al committente passa da **ASGARD**. **Restano da definire solo due cose:** come si gestisce **ingress/TLS** e dove/come si tengono i **segreti applicativi**. **Come si chiude:** CSI precisa questi due dettagli.

### ✅ Responsabile della migrazione dati (GOV-03) — CHIUSO 16/07/2026
**Cosa significa:** si passa da PostgreSQL 9 a 17; serve un piano scritto (CONSPREF-DMP) e qualcuno lato CSI che lo guidi. **Stato:** la redazione del CONSPREF-DMP è **in carico a CSI Piemonte**. Resta da produrre la bozza v1 (Sprint 0).

---

## 3. CDU-17 (lo "snapshot" PULL) — da concordare

### 🔴 I sistemi esterni possono chiamarci loro? (PULL-08)
**Cosa significa:** nel nuovo modello è il sistema esterno (SIA) che viene a prendersi i dati. Bisogna verificare che tecnicamente possano farlo (firewall, ecc.). **Come si chiude:** CSI/ASR confermano la capacità.

### 🔴 Lo snapshot blocca o no le scritture? (PULL-01)
**Cosa significa:** per fare una "foto" coerente dei consensi, o fermiamo un attimo le acquisizioni (semplice, micro-stop) oppure usiamo una marca temporale (zero stop, più complesso). **Come si chiude:** CSI sceglie la variante.

### 🔴 Come avvisiamo il sistema esterno? (PULL-02)
**Cosa significa:** quando registriamo un nuovo endpoint, avvisiamo il SIA via email e/o webhook? **Come si chiude:** CSI indica il canale.

### 🔴 Scrivere la specifica del CDU-17 (PULL-09)
**Cosa significa:** va definita la specifica REST del servizio (parametri, risposta, errori). **Come si chiude:** la scriviamo insieme a CSI.

---

## 4. Integrazioni con altri sistemi

### ✅ Documentazione di AURA + credenziali (INT-01 / ID-03) — CHIUSO
**Cosa significa:** AURA è l'anagrafe per cercare i pazienti; servono il WSDL dei servizi (FindProfiliAnagrafici, getProfiloSanitario) e le credenziali IRIS per DEV. **Stato (call 20/07/2026):** **nessun servizio nuovo**, sono **gli stessi già presenti nei file di properties**, credenziali IRIS di DEV **incluse**. Niente da chiedere.

### ✅ Gestione Deleghe via API-Piemonte (INT-02) — CHIUSO
**Cosa significa:** le deleghe si raggiungono tramite il portale API-Piemonte, operazione `getDelegantiService`. **Stato (call 20/07/2026):** **già integrato, nulla da fare.**

### ✅ Operazioni SOAP di BATCH-01 (BAT-01) — CONFERMATO
**Cosa significa:** BATCH-01 notifica i consensi alle ASR con **SRV-03** (acquisizioni) e **SRV-04** (revoche/annullamenti). **Stato (call 20/07/2026):** confermato — operazioni esistenti, **"solo da svecchiare e integrare la nuova logica"**. Resta solo il dettaglio dei nomi esatti dei campi.

### ✅ Chi crea le credenziali dei nuovi sistemi (SEC-03) — DELEGATO
**Cosa significa:** ogni sistema esterno ha un suo `client_id`; va deciso chi lo crea. **Stato (call 20/07/2026):** lo gestisce l'**API Manager**; è un **evento raro**, eventualmente integrabile in seguito con una **CR**. Nessuna tabella di mappatura lato prodotto in V1.0.

### 🟠 Cos'è "LIS" e come si integra (INT-03)
**Cosa significa:** "LIS" compare come terzo canale di acquisizione ma non è chiaro l'acronimo né come funziona. **Come si chiude:** CSI scioglie l'acronimo e dà le specifiche.

### 🟠 Accesso ai componenti grafici QUASAR (INT-04)
**Cosa significa:** QUASAR è la libreria UI di riferimento del CSI. **Come si chiude:** CSI dà accesso al repository.

### 🟠 Registrazione dei profili su PUA (ID-02)
**Cosa significa:** l'app di Back Office va registrata sul Configuratore con i due profili operatore. **Come si chiude:** CSI esegue la registrazione.

---

## 5. Punti tecnici da chiarire o comunicare

### ⚪ Ogni quanto gira BATCH-02? (BAT-02 / frequenza)
**Cosa significa:** il processo che gestisce le informative scadute: l'ipotesi è "una volta al giorno, notturno". **Stato (call 20/07/2026):** **non vincolante**, si concorda in seguito.

### 🟠 Da quale informativa si legge "annulla_consensi"? (SC67) — DOMANDA DA RIFORMULARE
**Cosa significa:** quando un'informativa scade, il flag che decide se i consensi diventano "annullati" o solo "scaduti" va letto dall'informativa **scaduta** o dalla **nuova**? **Stato (call 20/07/2026):** **CSI non ha capito la domanda** così com'era posta. La riformuliamo con un esempio concreto:
> *L'informativa A (annulla_consensi = NO) scade e viene sostituita dalla B (annulla_consensi = SI). Un consenso ATTIVO legato ad A deve diventare **SCADUTO** (leggiamo il flag di A) o **ANNULLATO** (leggiamo il flag di B)? E qual è la tabella/campo da cui leggere il flag al momento della storicizzazione?*
> Nel documento §6.13 (scaduta) e §7.2 SQL (nuova) non concordano: la risposta CSI scioglie il nodo e uniforma il testo.

### 🟠 Le ASR sanno che "SCADUTO" cambia significato? (BAT-03) — CALL DEDICATA
**Cosa significa:** nel nuovo sistema lo stato SCADUTO scatta in un momento diverso (asincrono). Le ASR che leggono questi dati devono adeguarsi. **Stato (call 20/07/2026):** CSI lo considera "componente da gestire" → è prevista una **call dedicata** per definirne la gestione e la comunicazione alle ASR.

### ⚪ Prestazioni e lista delle ASR (GOV-04 / API-04 / API-05)
**Cosa significa:** servono i target di prestazione (tempi di risposta, carico) e l'elenco delle ASR coinvolte con i referenti tecnici. **Stato (call 20/07/2026):** la **lista ASR** è **non vincolante**, smarcabile in seguito. Restano gli SLA/NFR prima del collaudo.

---

## In sintesi: come si chiudono

Quasi tutti i punti si chiudono in **tre modi**:

1. **CSI ci dà un accesso o una risorsa** (database, repository, credenziali) → richiesta formale.
2. **CSI ci dà documentazione tecnica** (metadata GASP, WSDL AURA/Deleghe, URL e firma token) → consegna documenti.
3. **CSI/Regione prende una decisione** (variante snapshot, scope, deroghe sul SRS) → si decide in riunione e si verbalizza.

