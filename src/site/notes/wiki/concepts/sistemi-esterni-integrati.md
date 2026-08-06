---
{"dg-publish":true,"permalink":"/wiki/concepts/sistemi-esterni-integrati/","title":"Sistemi Esterni Integrati","tags":["integrazione","soap","rest","aura","sia","notificatore","gestione-deleghe","pua","configuratore","lis","mf53","mf55","mf33"],"dg-note-properties":{"title":"Sistemi Esterni Integrati","aliases":["Sistemi Esterni Integrati"],"type":"concept","tags":["integrazione","soap","rest","aura","sia","notificatore","gestione-deleghe","pua","configuratore","lis","mf53","mf55","mf33"],"created":"2026-05-05","updated":"2026-07-20","sources":["2026-03-02-conspref-srs-v1-revised","2019-06-01-webservice-consenso-regionale-v03","2026-03-02-domande-srs-csi-v02"],"related":["[[Gestione Consensi - Applicativo]]","[[Architettura IaaS]]","[[CSI Piemonte]]","[[wiki/concepts/batch-processes\|Processi Batch — BATCH-01, BATCH-02, BATCH-03]]","[[GASP Salute]]","[[analysis-2026-05-14-risposte-mf-srs-v3]]"]}}
---


# Sistemi Esterni Integrati

Sistemi esterni con cui [[wiki/concepts/gestione-consensi-applicativo\|Gestione Consensi - Applicativo]] si integra nel TO-BE. Tutti gestiti da [[wiki/entities/csi-piemonte\|CSI Piemonte]] o dalle ASR della Regione Piemonte.

---

## AURA — Anagrafe Unica Regionale Assistiti

| Aspetto                    | Dettaglio                                                            |
| -------------------------- | -------------------------------------------------------------------- |
| Gestore                    | [[wiki/entities/csi-piemonte\|CSI Piemonte]]                                       |
| Protocollo                 | SOAP + WS-Security UsernameToken (credenziali IRIS)                  |
| Credenziali                | IRIS — ✅ **incluse nei file di properties** (call CSI 20/07/2026)    |
| Uso                        | Ricerca assistito per codice fiscale (CDU-07, CDU-08 area Operatore) |
| Servizi invocati (MF53R52) | **FindProfiliAnagrafici**, **getProfiloSanitario**                   |
| Fallback                   | **NESSUNO** — vedi nota sotto                                        |
| WSDL                       | ✅ **Nessun servizio nuovo** — gli stessi già nei properties (call CSI 20/07/2026) |

> ✅ **Chiuso (call CSI 20/07/2026 — INT-01/ID-03):** CSI conferma che **non ci saranno nuovi servizi**, sono **sempre gli stessi già presenti nei file di properties**; le **credenziali IRIS per DEV** sono incluse. Niente da richiedere.

### Comportamento ricerca assistito (MF53R52, MF55R54)

> Il sistema invoca **FindProfiliAnagrafici** + **getProfiloSanitario** di AURA. Se il CF non viene trovato → messaggio fisso: **«La ricerca con il filtro fornito non ha prodotto risultati»**. **Nessuna chiamata a SistemaTS.**

> ⚠️ **Eliminazione SistemaTS dal SRS:** decisione MF55R54 elimina **tutti i riferimenti** a SistemaTS come fallback in SRS §6.7. SistemaTS non fa parte dell'integrazione del sistema Gestione Consensi TO-BE.

Conseguenza wiki: rimosso dal corpus dei sistemi integrati. La voce SistemaTS NON deve comparire nei diagrammi di contesto, nelle mappe di integrazione né nel modello dati.

---

## SIA ASR — Sistemi Informativi Aziendali delle ASR

Sistema bidirezionale — consumatore API REST e destinatario notifiche SOAP.

| Direzione | Protocollo | CDU/Batch | Stato |
|---|---|---|---|
| Inbound (SIA→Regionale) | REST OpenAPI 3.x — OAuth2 `client_credentials` via **API Manager CSI (APIMBBONE)** (07/2026) | CDU-15, CDU-16, **CDU-17 (snapshot)**, **servizi endpoint CRUD** | CDU-15/16: [[wiki/analyses/analysis-2026-05-06-openapi-cdu-15-16\|v0.1-DRAFT prodotta]], 5 TBD CSI; **CDU-17 rielaborato e confermato (call 20/07/2026)** — swagger da produrre |
| Outbound (Regionale→SIA) | SOAP AS-IS invariato — [[wiki/sources/2019-06-01-webservice-consenso-regionale-v03\|Specifica WebService ConsensoRegionaleAziendale v03 (AS-IS)]] | BATCH-01 (solo notifiche puntuali) | Contratto definito ✅ |

**BATCH-03 push rimosso:** l'allineamento massivo per nuovo endpoint passa a modello PULL (CDU-17). Vedi [[wiki/concepts/alternativa-batch-03-pull\|Alternativa BATCH-03 — PULL CDU-17 (centro stella)]].

**Sicurezza outbound SOAP (BATCH-01):** WS-Security X509, un certificato per ASR — richiederli in Sprint 0.
**Sicurezza inbound REST:** OAuth2 Client Credentials via **API Manager APIMBBONE**; il Gateway inoltra al backend il **`codice_ente`** (isolamento per ente). **`cons_t_client_ente` fuori scope V1.0** (estensione futura). Vedi [[wiki/concepts/sicurezza-cdu-15-16\|Sicurezza CDU-15-16 — Modello Autorizzazione per Ente]].

**Rischi aperti:** ambiguità BATCH-01 WSDL + semantica SCADUTO AS-IS≠TO-BE. Vedi [[wiki/concepts/batch-processes\|Processi Batch — BATCH-01, BATCH-02, BATCH-03]].

### CDU-17 — Contratto SIA come Caller (modello PULL)

> ✅ **Rielaborato e confermato (call CSI 20/07/2026).** Diagrammi aggiornati: [[CDU-17_diagramma-sequenza\|CDU-17 — Allineamento PULL]] · [[Manutenzione-endpoint_diagramma-sequenza\|Manutenzione endpoint]]. Il SIA è **chiamante attivo** (confermato); i servizi transitano dall'**API Gateway APIMBBONE**. Oltre allo snapshot, sono esposti alle aziende i **servizi endpoint CRUD** (inserimento/modifica/eliminazione) e lo **scenario di manutenzione** (stato `IN_MANUTENZIONE`). Le operazioni della **web app** restano dirette (senza API Manager).

**Inversione del contratto:** nel modello PULL, SIA non è destinatario passivo ma **chiamante attivo**. Il sistema Gestione Consensi espone un endpoint REST paginato che SIA invoca per ottenere lo snapshot dei consensi.

**Flusso PULL (autoritativo):**

```
SIA ASR  →  PATCH /api/v1/endpoints/{endp_id}/stato-allineamento { IN_CORSO }   (blocco obbligatorio)
         →  loop: GET /api/v1/consensi/snapshot?codice_ente&codice_consenso&cursor&page_size
                        ↓  (via API Gateway APIMBBONE — codice_ente inoltrato)
         [EnteAuthorizationFilter → WHERE codice_ente]
                        ↓
         Gestione Consensi  →  200 { items[], next_cursor, has_more }
         →  PATCH .../stato-allineamento { COMPLETATO }   (sblocco)
         →  passo 5: SIA notifica alla webapp COMPLETATO + dati ultimo invio (canalità PULL-02)
```

**Parametri della chiamata snapshot SIA→Regionale:**

| Parametro | Tipo | Note |
|---|---|---|
| `codice_ente` | string | Codice ASR — **coerenza verificata contro il `codice_ente` inoltrato dal Gateway APIM** |
| `codice_consenso` | string | Sotto-tipologia di consenso in allineamento |
| `cursor` | string (opaco) | Cursore base64 per la pagina successiva (no offset) |
| `page_size` | int | Dimensione pagina (default 1000) |
| Authorization header | Bearer JWT | OAuth2 Client Credentials via API Manager APIMBBONE — stesso schema CDU-15/16 |

**Applicazione [[wiki/concepts/sicurezza-cdu-15-16\|EnteAuthorizationFilter]] su chiamata inbound:**

1. **Token** validato dall'**API Manager (Key Manager)**; il Gateway inoltra al backend il **`codice_ente`** (e il CF da Shibboleth)
2. **Livello ente:** il repository forza `WHERE codice_ente = :authorizedEnte` (ente del Gateway) → SIA vede **solo** i consensi del proprio ente
3. In V1.0 **nessun lookup** su `cons_t_client_ente` (tabella fuori scope, estensione futura)

> ✅ **Gap chiuso (20/07/2026):** la capacità del SIA di chiamare (PULL-08) è confermata e il modello di sicurezza inbound è definito (isolamento su `codice_ente` del Gateway). **Resta da produrre lo swagger (OpenAPI) di CDU-17.**

**Differenze rispetto al modello PUSH (BATCH-03 AS-IS):**

| Aspetto | PUSH (AS-IS) | PULL (TO-BE CDU-17) |
|---|---|---|
| Iniziatore | Gestione Consensi (batch scheduler) | SIA ASR |
| Frequenza | Schedulata internamente | Definita da SIA (TBD con CSI) |
| Scope dati | Tutti i delta dal batch precedente | Delta da `from` timestamp |
| Autorizzazione | Interna — decidiamo noi cosa spedire | SIA chiede → noi filtriamo |
| Spec contratto | WSDL AS-IS disponibile ✅ | Modello confermato ✅ — **swagger (OpenAPI) CDU-17 da produrre** |

**Relazione SIA:** 1 a n — un'azienda ASR può avere **più sistemi** SIA. Se uno di questi è in manutenzione, CSI interrompe l'invio verso tutti i sistemi dell'azienda. Vedi [[wiki/concepts/batch-processes\|Processi Batch]] §Gestione Manutenzione ASR.

**Gap aperti prima di andare in produzione:**

| Gap | Responsabile | Stato |
|---|---|---|
| Swagger (OpenAPI) CDU-17 (endpoint, payload, error codes, passo 5, endpoint CRUD, manutenzione) | Exprivia + CSI | 🟠 **Residuo attivo** |
| ~~Conferma che SIA supporta chiamate PULL attive~~ | CSI Piemonte | ✅ Confermato (PULL-08, call 20/07/2026) |
| `EnteAuthorizationFilter` per caller esterno | Exprivia | ✅ Definito — isolamento su `codice_ente` del Gateway |
| Security scheme CDU-17 nello YAML OpenAPI | Exprivia | 🟠 Da aggiungere nello swagger CDU-17 |
| Frequenza di polling SIA (SLA snapshot) | CSI Piemonte | ⚪ Da precisare (non bloccante) |

---

## Notificatore UNP — User Notification Platform

| Aspetto           | Dettaglio                                                                                                                    |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Gestore           | [[wiki/entities/csi-piemonte\|CSI Piemonte]]                                                                                               |
| Protocollo        | REST                                                                                                                         |
| Canali supportati | Email, push, AppIO                                                                                                           |
| Uso               | Notifiche applicative generiche verso cittadino (NON per conferma rilascio consenso — quella va via Notificatore di Deleghe) |
| Documentazione    | gitlab.csi.it/user-notification-platform/unpdocumentazione                                                                   |

---

## Notificatore di Deleghe (MF33R31) — distinto da UNP

> ✅ **Chiarito (call CSI 06/08/2026):** componente **AS-IS legacy, riciclato** — nessun nuovo sviluppo richiesto. Aggiorna la riga "❌ Da richiedere" precedente in §Stato approvvigionamento sotto.

| Aspetto     | Dettaglio                                                                    |
| ----------- | ---------------------------------------------------------------------------- |
| Gestore     | [[wiki/entities/csi-piemonte\|CSI Piemonte]]                                               |
| Uso         | **Notifica cittadino/delegato post-acquisizione consenso**                   |
| Timing      | Parte **SOLO dopo conferma notifica aziende** (BATCH-01, stato = COMPLETATO) |
| Distinzione | Servizio separato dall'UNP — non confondere                                  |
| Sviluppo    | **AS-IS legacy, riciclato — nessun nuovo sviluppo** (call CSI 06/08/2026)    |

> ⚠️ **Importante:** Notificatore di Deleghe ≠ Notificatore UNP. UNP per notifiche applicative generiche; Notificatore di Deleghe per la conferma al cittadino/delegato a valle dell'acquisizione consenso. Documentare distinzione in SRS §3 e §7 (sezione notifiche).

---

## Gestione Deleghe ❌ OUT scope

> 🔴 **Fuori dal perimetro di sviluppo** (call CSI 06/08/2026, [[wiki/docs/adr/ADR-021-perimetro-solo-operatore\|ADR-021]]) — l'uso documentato è dal pulsante "Deleghe" della Webapp Cittadino (MF20R19), non un deliverable di questo progetto. Sezione mantenuta come contesto storico.

| Aspetto            | Dettaglio                                                                                                       |
| ------------------ | --------------------------------------------------------------------------------------------------------------- |
| Protocollo         | SOAP                                                                                                            |
| Operazione SOAP    | **`getDelegantiService`**                                                                                       |
| Routing            | **via API-Piemonte** (API Manager) — non chiamata diretta                                                       |
| Accreditamento     | Tramite **portale API-Piemonte** (richiesto da Exprivia a CSI)                                                  |
| Uso                | Verifica deleghe familiari — cittadino che opera per conto di un delegante (pulsante "Deleghe" webapp, MF20R19) |
| WSDL               | ✅ **Già integrato — nulla da fare** (call CSI 20/07/2026)                                                       |
| Stato produzione   | Scenario delegante già attivo in produzione (MF22R21)                                                           |

> ✅ **Chiuso (call CSI 20/07/2026 — INT-02):** l'integrazione con Gestione Deleghe (`getDelegantiService` via API-Piemonte) è **già integrata, non c'è nulla da fare**. Cadono l'accreditamento portale e la firma del token come punti aperti.
>
> ✅ **Riconfermato (call CSI 06/08/2026):** componente **AS-IS legacy, riciclato integralmente** — nessun nuovo sviluppo richiesto, indipendentemente dal perimetro di scope. Resta comunque fuori dal perimetro di sviluppo di questo progetto perché l'unico uso documentato è dal pulsante "Deleghe" della Webapp Cittadino (vedi banner sopra).

**Flusso di integrazione (da immagine DelegheApi — verbale 11/06/2026):**

```
Gestione Consensi  →  getDelegantiService  →  API-Piemonte  →  DelegheApi
                                              (accred. portale)
```

> Questo è il pattern "bridge API Manager" menzionato nel verbale 11/06/2026. Gestione Consensi sarà accreditata sul portale API-Piemonte; la chiamata transita per l'API Manager che poi raggiunge DelegheApi. Il meccanismo di firma del token usato da API-Piemonte è ancora punto aperto (vedi [[wiki/concepts/sicurezza-cdu-15-16\|Sicurezza CDU-15-16]] §1.2).

Variante errore [PROPOSTA]: se il servizio non risponde, il sistema impedisce la selezione del delegante e mostra avviso. Il cittadino opera solo per sé stesso.

---

## LIS — Laboratorio (integrazione BE esistente, MF3R1/MF4R1 — rivisto ADR-020)

> ⚠️ **Rivisto (call CSI 06/08/2026, chiude INT-03):** LIS **non è un canale di acquisizione UI dedicato**. CSI ha chiarito che, sui nuovi sviluppi funzionali e non funzionali, **non esiste un terzo canale di acquisizione consensi** — l'acquisizione presso LIS (e sistemi analoghi come RIS) avviene tramite un'**integrazione BE già presente nel codice sorgente AS-IS**. Attività TO-BE: individuare, verificare e migrare tale integrazione al nuovo stack. Vedi [[wiki/docs/adr/ADR-020-lis-integrazione-be-esistente\|ADR-020]] (supersede [[wiki/docs/adr/ADR-017-lis-terzo-canale\|ADR-017]]).

| Aspetto | Dettaglio |
|---|---|
| Funzione | Acquisizione/sincronizzazione consenso presso il laboratorio, via integrazione backend |
| Integrazione | **Già esistente nel codice sorgente AS-IS** — da individuare (protocollo/payload) e migrare al nuovo stack, non da progettare ex-novo |
| Implicazione SRS | Diagramma di contesto §1/§2: **2 canali di acquisizione** (Cittadino, Operatore); LIS compare come sistema esterno integrato, non come canale |

Nota: l'acronimo LIS resta da chiarire formalmente con CSI, ma non è più bloccante per la progettazione (non blocca più INT-03, chiuso il 06/08/2026).

---

## Configuratore Regionale / PUA

> ✅ **Chiarito (call CSI 06/08/2026):** **un solo profilo Operatore** in PUA/Configuratore, che copre **tutte** le nuove funzionalità da sviluppare (CDU-01a, CDU-05, CDU-07÷CDU-14). **Nessun nuovo profilo da creare.** Corregge la precedente ipotesi di "2 profili operatore" (Operatore Sanitario/Amministrativo + Back Office, o Operatore + Amministratore a seconda del documento — nomenclatura mai stata univoca in wiki).

| Sistema | Funzione | Profilo utente |
|---|---|---|
| PUA (Punto Unico di Accesso) | Autenticazione operatore | RUPAR / IRIDE |
| Configuratore Regionale | Censimento applicazioni, profilo operatore unico, endpoint ASR | **1 profilo Operatore** (unico, copre CDU-01a/05/07÷14) |

> **Nota MF7R5:** Cittadino **NON** è profilo applicativo del Configuratore — accede via webapp dedicata SPID/CIE (comunque fuori scope, [[wiki/docs/adr/ADR-021-perimetro-solo-operatore\|ADR-021]]). Configuratore gestisce il profilo operatore.

Registrazione app PUA (**1 profilo Operatore** — non 2, call CSI 06/08/2026) da richiedere a [[wiki/entities/csi-piemonte\|CSI Piemonte]] in Sprint 0 ❌.

**Servizio PUA post-autenticazione (SRS v3 §2.3):** dopo il login via PUA, il backend invoca `getTokenInformation2` per leggere il profilo dell'operatore autenticato e adattare dinamicamente l'interfaccia (mostra solo sezioni/CDU abilitati per quel profilo). Un operatore può essere abilitato a entrambi i profili contemporaneamente.

---

## Mappa integrazioni sintetica (aggiornata)

```
[Cittadino] → GASP Salute (SAML2 via Shibboleth SP) → Webapp Citt   ❌ OUT scope (ADR-021, 06/08/2026)
[Operatore] → PUA/Configuratore Regionale → Webapp Operatore
                              ↓
[Applicativo Gestione Consensi]
        ├─ AURA              (SOAP/IRIS) — FindProfiliAnagrafici, getProfiloSanitario
        ├─ Gestione Deleghe  (SOAP/OAuth2) — verifica deleghe familiari [❌ OUT scope, uso solo Webapp Citt — ADR-021]
        ├─ Notificatore Deleghe (REST?) — conferma post-acquisizione al cittadino
        ├─ Notificatore UNP  (REST) — notifiche applicative generiche
        ├─ SIA ASR           (SOAP outbound) ← BATCH-01 notifiche puntuali
        ├─ SIA ASR           (REST inbound) ← CDU-15, CDU-16 [spec v0.1-DRAFT]
        ├─ SIA ASR           (REST inbound via API Manager) ← CDU-17 PULL snapshot [✅ confermato 20/07 — swagger da produrre]
        └─ LIS (e analoghi es. RIS) — integrazione BE esistente AS-IS, da individuare e migrare [ADR-020, 06/08/2026]
```

> Nota: LIS **non** è più un ramo di accesso UI (`[Cittadino c/o LIS] → operatore LIS → Webapp Operatore`) come da ADR-017 originario — è un'integrazione lato backend, elencata sotto l'applicativo Gestione Consensi al pari degli altri sistemi esterni. Vedi [[wiki/docs/adr/ADR-020-lis-integrazione-be-esistente\|ADR-020]].

---

## Stato approvvigionamento Sprint 0

| Sistema                  | Cosa serve                                                                        | Stato                                                                                                                                    |
| ------------------------ | --------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| AURA                     | Credenziali IRIS + WSDL (FindProfiliAnagrafici, getProfiloSanitario)              | ✅ Nei properties (call 20/07/2026) — nessun servizio nuovo                                                                               |
| SIA ASR                  | Certificati X509 per ogni ASR (AS-IS) / credenziali OAuth via APIMBBONE (TO-BE)   | ❌ Da richiedere (AS-IS) · TO-BE gestito da APIM                                                                                          |
| Gestione Deleghe ❌ OUT   | WSDL                                                                              | ✅ Già integrato (call 20/07/2026) — fuori scope di sviluppo (ADR-021, 06/08/2026), uso solo Webapp Cittadino                             |
| Notificatore di Deleghe  | API + integration spec                                                            | ✅ **AS-IS legacy, riciclato — nessun nuovo sviluppo** (call CSI 06/08/2026; era "❌ Da richiedere" prima di questa call, distinta da UNP) |
| Notificatore UNP         | Già documentato — gitlab.csi.it                                                   | ✅ Riferimento disponibile                                                                                                                |
| PUA                      | Registrazione **1 profilo Operatore** (era "2 profili", corretto call CSI 06/08/2026) | ❌ Da richiedere                                                                                                                      |
| LIS (e analoghi es. RIS) | Individuare integrazione BE esistente nel sorgente AS-IS + migrare al nuovo stack | ❌ Da individuare — dipende da accesso repo/DB AS-IS (INT-04, TECH-01)                                                                    |
| GASP Salute ❌ OUT        | ~~Documentazione protocollo OIDC/SAML2~~                                          | ⚪ Non più necessario — fuori scope di sviluppo (ADR-021, 06/08/2026)                                                                     |
| SistemaTS                | —                                                                                 | ⛔ **NON integrato** (decisione MF55R54 — rimosso dal SRS)                                                                                |

---

## ADR correlati

| ADR | Decisione |
|---|---|
| [ADR-009](ADR-009-eliminazione-sistemats.md) | Eliminazione SistemaTS dall'integrazione |
| [ADR-012](ADR-012-notificatore-deleghe-post-completato.md) | Notificatore di Deleghe ≠ Notificatore UNP, post-COMPLETATO |
| [ADR-014](ADR-014-apache-cxf-soap-client.md) | Apache CXF client SOAP (AURA, Deleghe, SIA outbound) |
| [ADR-021](ADR-021-perimetro-solo-operatore.md) | Perimetro progetto: solo Webapp Operatore — Gestione Deleghe/GASP fuori scope |
| [ADR-020](ADR-020-lis-integrazione-be-esistente.md) | LIS/RIS integrazione BE esistente (supersede [ADR-017](ADR-017-lis-terzo-canale.md)) |
