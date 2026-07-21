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
| Inbound (SIA→Regionale) | REST OpenAPI 3.x — OAuth2 `client_credentials` via **API Manager CSI (APIMBBONE)** (07/2026) | CDU-15, CDU-16, **CDU-17 (snapshot)** | CDU-15/16: [[wiki/analyses/analysis-2026-05-06-openapi-cdu-15-16\|v0.1-DRAFT prodotta]], 5 TBD CSI; CDU-17 in proposta TR68 |
| Outbound (Regionale→SIA) | SOAP AS-IS invariato — [[wiki/sources/2019-06-01-webservice-consenso-regionale-v03\|Specifica WebService ConsensoRegionaleAziendale v03 (AS-IS)]] | BATCH-01 (solo notifiche puntuali) | Contratto definito ✅ |

**BATCH-03 push rimosso:** l'allineamento massivo per nuovo endpoint passa a modello PULL (CDU-17). Vedi [[wiki/concepts/alternativa-batch-03-pull\|Alternativa BATCH-03 — PULL CDU-17 (centro stella)]].

**Sicurezza outbound SOAP (BATCH-01):** WS-Security X509, un certificato per ASR — richiederli in Sprint 0.
**Sicurezza inbound REST:** OAuth2 Client Credentials + tabella `cons_t_client_ente`. Vedi [[wiki/concepts/sicurezza-cdu-15-16\|Sicurezza CDU-15-16 — Modello Autorizzazione per Ente]].

**Rischi aperti:** ambiguità BATCH-01 WSDL + semantica SCADUTO AS-IS≠TO-BE. Vedi [[wiki/concepts/batch-processes\|Processi Batch — BATCH-01, BATCH-02, BATCH-03]].

### CDU-17 — Contratto SIA come Caller (modello PULL)

> ⚠️ **Inversione del contratto:** nel modello PULL, SIA non è più destinatario passivo ma **chiamante attivo**. Il sistema Gestione Consensi espone un endpoint REST che SIA invoca periodicamente per ottenere uno snapshot dei consensi aggiornati.

**Flusso PULL:**

```
SIA ASR  →  GET /cdu-17/snapshot?ente={codice_ente}&from={timestamp}
                        ↓
         [EnteAuthorizationFilter — 3 livelli]
                        ↓
         Gestione Consensi  →  risponde con payload consensi
```

**Parametri attesi nella chiamata SIA→Regionale:**

| Parametro | Tipo | Note |
|---|---|---|
| `codice_ente` | string | Codice ASR richiedente — validato contro `cons_t_client_ente` |
| `from` | ISO 8601 timestamp | Data di inizio finestra snapshot (delta, non full dump) |
| Authorization header | Bearer JWT | OAuth2 Client Credentials via API Manager APIMBBONE — stesso schema CDU-15/16 |

**Applicazione [[wiki/concepts/sicurezza-cdu-15-16\|EnteAuthorizationFilter]] su chiamata inbound:**

Il filtro opera a 3 livelli su ogni richiesta SIA:
1. **Livello client:** JWT validato → `client_id` mappato a `codice_ente` in `cons_t_client_ente`
2. **Livello ente:** la risposta include **solo** i consensi dell'ente corrispondente al `client_id` — SIA non può richiedere dati di enti diversi dal proprio
3. **Livello dato:** eventuali consensi di assistiti senza ASR di riferimento non vengono esposti

> ⚠️ **Gap critico:** il filtro è progettato per richieste interne/webapp. La sua applicazione su chiamate entranti da sistema terzo (SIA) non è documentata nella spec [[wiki/concepts/sicurezza-cdu-15-16\|CDU-15-16]] né nello YAML [[wiki/analyses/analysis-2026-05-06-openapi-cdu-15-16\|OpenAPI v0.1]]. La spec CDU-17 non esiste ancora.

**Differenze rispetto al modello PUSH (BATCH-03 AS-IS):**

| Aspetto | PUSH (AS-IS) | PULL (TO-BE CDU-17) |
|---|---|---|
| Iniziatore | Gestione Consensi (batch scheduler) | SIA ASR |
| Frequenza | Schedulata internamente | Definita da SIA (TBD con CSI) |
| Scope dati | Tutti i delta dal batch precedente | Delta da `from` timestamp |
| Autorizzazione | Interna — decidiamo noi cosa spedire | SIA chiede → noi filtriamo |
| Spec contratto | WSDL AS-IS disponibile ✅ | **Spec REST CDU-17 non scritta ❌** |

**Relazione SIA:** 1 a n — un'azienda ASR può avere **più sistemi** SIA. Se uno di questi è in manutenzione, CSI interrompe l'invio verso tutti i sistemi dell'azienda. Vedi [[wiki/concepts/batch-processes\|Processi Batch]] §Gestione Manutenzione ASR.

**Gap aperti prima di andare in produzione:**

| Gap | Responsabile | Stato |
|---|---|---|
| Spec REST CDU-17 (endpoint, payload, error codes) | Exprivia + CSI | ❌ Non scritta |
| Conferma CSI che SIA supporta chiamate PULL attive | CSI Piemonte | ❌ Punto aperto (vedi [[wiki/analyses/analysis-2026-05-14-punti-aperti-csi\|Punti Aperti CSI]]) |
| Estensione `EnteAuthorizationFilter` per caller esterno | Exprivia | ❌ Non documentata |
| Aggiunta security scheme CDU-17 nello YAML OpenAPI | Exprivia | ❌ YAML attuale copre solo CDU-15/16 |
| Frequenza di polling SIA (SLA snapshot) | CSI Piemonte | ❌ TBD |

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

| Aspetto     | Dettaglio                                                                    |
| ----------- | ---------------------------------------------------------------------------- |
| Gestore     | [[wiki/entities/csi-piemonte\|CSI Piemonte]]                                               |
| Uso         | **Notifica cittadino/delegato post-acquisizione consenso**                   |
| Timing      | Parte **SOLO dopo conferma notifica aziende** (BATCH-01, stato = COMPLETATO) |
| Distinzione | Servizio separato dall'UNP — non confondere                                  |

> ⚠️ **Importante:** Notificatore di Deleghe ≠ Notificatore UNP. UNP per notifiche applicative generiche; Notificatore di Deleghe per la conferma al cittadino/delegato a valle dell'acquisizione consenso. Documentare distinzione in SRS §3 e §7 (sezione notifiche).

---

## Gestione Deleghe

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

**Flusso di integrazione (da immagine DelegheApi — verbale 11/06/2026):**

```
Gestione Consensi  →  getDelegantiService  →  API-Piemonte  →  DelegheApi
                                              (accred. portale)
```

> Questo è il pattern "bridge API Manager" menzionato nel verbale 11/06/2026. Gestione Consensi sarà accreditata sul portale API-Piemonte; la chiamata transita per l'API Manager che poi raggiunge DelegheApi. Il meccanismo di firma del token usato da API-Piemonte è ancora punto aperto (vedi [[wiki/concepts/sicurezza-cdu-15-16\|Sicurezza CDU-15-16]] §1.2).

Variante errore [PROPOSTA]: se il servizio non risponde, il sistema impedisce la selezione del delegante e mostra avviso. Il cittadino opera solo per sé stesso.

---

## LIS — Laboratorio (canale acquisizione consensi) (MF3R1, MF4R1)

| Aspetto | Dettaglio |
|---|---|
| Funzione | Acquisizione consenso del cittadino presso il laboratorio (front-office) |
| Integrazione | Da definire — canale aggiuntivo rispetto a webapp Citt e webapp Operatore |
| Implicazione SRS | Diagramma di contesto §1/§2 deve riflettere 3 canali, non 2 |

Nota: l'acronimo LIS va chiarito formalmente con CSI nella prossima revisione SRS.

---

## Configuratore Regionale / PUA

| Sistema | Funzione | Profilo utente |
|---|---|---|
| PUA (Punto Unico di Accesso) | Autenticazione operatori sanitari/amministrativi | RUPAR / IRIDE |
| Configuratore Regionale | Censimento applicazioni, profili operatori, endpoint ASR | Back Office |

> **Nota MF7R5:** Cittadino **NON** è profilo applicativo del Configuratore — accede via webapp dedicata SPID/CIE. Configuratore gestisce solo i profili operatore.

Registrazione app PUA (2 profili operatore) da richiedere a [[wiki/entities/csi-piemonte\|CSI Piemonte]] in Sprint 0 ❌.

**Servizio PUA post-autenticazione (SRS v3 §2.3):** dopo il login via PUA, il backend invoca `getTokenInformation2` per leggere il profilo dell'operatore autenticato e adattare dinamicamente l'interfaccia (mostra solo sezioni/CDU abilitati per quel profilo). Un operatore può essere abilitato a entrambi i profili contemporaneamente.

---

## Mappa integrazioni sintetica (aggiornata)

```
[Cittadino] → GASP Salute (SAML2 via Shibboleth SP) → Webapp Citt
[Cittadino c/o LIS] → operatore LIS → Webapp Operatore (caso d'uso LIS)
[Operatore] → PUA/Configuratore Regionale → Webapp Operatore
                              ↓
[Applicativo Gestione Consensi]
        ├─ AURA              (SOAP/IRIS) — FindProfiliAnagrafici, getProfiloSanitario
        ├─ Gestione Deleghe  (SOAP/OAuth2) — verifica deleghe familiari
        ├─ Notificatore Deleghe (REST?) — conferma post-acquisizione al cittadino
        ├─ Notificatore UNP  (REST) — notifiche applicative generiche
        ├─ SIA ASR           (SOAP outbound) ← BATCH-01 notifiche puntuali
        ├─ SIA ASR           (REST inbound) ← CDU-15, CDU-16 [spec v0.1-DRAFT]
        └─ SIA ASR           (REST inbound) ← CDU-17 PULL snapshot [⚠️ spec mancante — SIA come caller]
```

---

## Stato approvvigionamento Sprint 0

| Sistema | Cosa serve | Stato |
|---|---|---|
| AURA | Credenziali IRIS + WSDL (FindProfiliAnagrafici, getProfiloSanitario) | ✅ Nei properties (call 20/07/2026) — nessun servizio nuovo |
| SIA ASR | Certificati X509 per ogni ASR (AS-IS) / credenziali OAuth via APIMBBONE (TO-BE) | ❌ Da richiedere (AS-IS) · TO-BE gestito da APIM |
| Gestione Deleghe | WSDL | ✅ Già integrato (call 20/07/2026) |
| Notificatore di Deleghe | API + integration spec | ❌ Da richiedere (distinta da UNP) |
| Notificatore UNP | Già documentato — gitlab.csi.it | ✅ Riferimento disponibile |
| PUA | Registrazione 2 profili applicativo | ❌ Da richiedere |
| LIS | Specifica integrazione canale acquisizione | ❌ Da chiarire con CSI (acronimo + spec) |
| GASP Salute | ~~Documentazione protocollo OIDC/SAML2~~ | ✅ SAML2 confermato (verbale 11/06/2026); metadata SP ricevuti (TEST/preprod, 07/2026) — resta censimento federazione (`identita.federazione@csi.it`) |
| SistemaTS | — | ⛔ **NON integrato** (decisione MF55R54 — rimosso dal SRS) |

---

## ADR correlati

| ADR | Decisione |
|---|---|
| [ADR-009](ADR-009-eliminazione-sistemats.md) | Eliminazione SistemaTS dall'integrazione |
| [ADR-012](ADR-012-notificatore-deleghe-post-completato.md) | Notificatore di Deleghe ≠ Notificatore UNP, post-COMPLETATO |
| [ADR-014](ADR-014-apache-cxf-soap-client.md) | Apache CXF client SOAP (AURA, Deleghe, SIA outbound) |
| [ADR-017](ADR-017-lis-terzo-canale.md) | LIS terzo canale di acquisizione |
