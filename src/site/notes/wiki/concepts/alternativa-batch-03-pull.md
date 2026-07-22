---
{"dg-publish":true,"permalink":"/wiki/concepts/alternativa-batch-03-pull/","title":"Alternativa BATCH-03 — PULL CDU-17 (centro stella)","tags":["batch-03","cdu-17","pull","centro-stella","allineamento","sia","openapi","tr34","proposta"],"dg-note-properties":{"title":"Alternativa BATCH-03 — PULL CDU-17 (centro stella)","aliases":["Alternativa BATCH-03 — PULL CDU-17 (centro stella)"],"type":"concept","tags":["batch-03","cdu-17","pull","centro-stella","allineamento","sia","openapi","tr34","proposta"],"created":"2026-05-14","updated":"2026-07-20","sources":["2026-03-02-conspref-srs-v1-revised"],"related":["[[wiki/concepts/batch-processes\|Processi Batch — BATCH-01, BATCH-02, BATCH-03]]","[[wiki/concepts/sicurezza-cdu-15-16\|Sicurezza CDU-15-16 — Modello Autorizzazione per Ente]]","[[wiki/analyses/analysis-2026-05-06-openapi-cdu-15-16\|analysis-2026-05-06-openapi-cdu-15-16]]","[[Sistemi Esterni Integrati]]","[[wiki/concepts/ciclo-vita-consenso\|Ciclo di Vita del Consenso]]","[[Gestione Consensi - Applicativo]]","[[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|analysis-2026-05-14-risposte-mf-srs-v3]]"]}}
---


# Alternativa BATCH-03 — PULL CDU-17 (centro stella)

**Origine:** risposta tecnica al commento cliente **TR34** sulla revisione SRS bozza v3 (PDF righe 4951–4956, sez. §7.3 BATCH-03). Rinumerato come **TR68** nella revisione `CONSPREF-SRS-V1.0-revised_bozza_v3_CSI_lavorazione.pdf` (MF69R68). Vedi mapping in [[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|analysis-2026-05-14-risposte-mf-srs-v3]].

**Quesito originale TR34 (cliente CSI/Regione):**
> "No, allineamento massivo deve avvenire con un passaggio di dati all'interno dell'azienda, altrimenti ci carichiamo di un onere che non ci è dovuto. Vogliamo spingere verso un centro stella!!!!! Or export dei dati ma con interruzione del servizio online fino al caricamento."

**Status:** ✅ **CONFERMATO dal committente (call CSI 20/07/2026)** — CDU-17 rielaborato per intero come da mail cliente.

> ✅ **Rielaborazione CDU-17 — call CSI 20/07/2026.** Il committente ha fornito il rifacimento completo del caso d'uso. Novità rispetto alla prima proposta:
> - **Attore** = **operatore con profilo Back Office** sulla **web app Gestione Consensi** (accesso da **PUA**), non un "operatore CSI" generico. Scenario tipico: azienda con ≥1 endpoint già attivo (es. RIS) che ne attiva un altro (es. LIS) → serve allineamento.
> - **Variante B (watermark / no-blocco) ELIMINATA.** Resta **solo la Variante A**: il blocco temporaneo delle acquisizioni (`IN_CORSO`) è **obbligatorio** per garantire uno snapshot coerente.
> - **Nuovo passo 5:** a chiusura, il SIA **comunica alla webapp lo stato `COMPLETATO` + i dati dell'ultimo invio andato a buon fine**, tramite la **canalità PULL-02**.
> - **PULL-02 (canale notifica) DECISO:** email **e/o** webhook, via **parametro di configurazione** (solo email / solo webhook / entrambi). Con il **webhook** il SIA **espone un servizio REST** il cui **contratto, firma e sicurezza sono forniti da CSI**.
> - **PULL-01 DECISO** (blocco obbligatorio), **PULL-08 DECISO** (il SIA **può** chiamare i servizi REST).
> - **Servizi endpoint CRUD** (inserimento/modifica/eliminazione endpoint) della web app **esposti anche alle aziende via API Manager (APIMBBONE)**; le operazioni della **web app NON passano dall'API Manager** (doppia esposizione).
> - **Nuovo scenario di manutenzione endpoint** (senza inserimento di nuovo endpoint): stato **`IN_MANUTENZIONE`**, blocco acquisizione consensi + notifica webapp di blocco/ripresa (`COMPLETATO` + dati ultimo invio). Diagramma dedicato: [[Manutenzione-endpoint_diagramma-sequenza\|Manutenzione endpoint]].
> - Sicurezza allineata al modello 07/2026: token via **API Manager APIMBBONE**, `codice_ente` inoltrato dal Gateway; **`cons_t_client_ente` fuori scope V1.0** (cfr. [[wiki/concepts/sicurezza-cdu-15-16\|Sicurezza CDU-15/16]]).
>
> Diagrammi di sequenza aggiornati: [[CDU-17_diagramma-sequenza\|CDU-17 — Allineamento PULL]] · [[Manutenzione-endpoint_diagramma-sequenza\|Manutenzione endpoint]].

---

## 1. Sintesi (TL;DR)

**Soluzione semplice e indolore:** sostituire **BATCH-03 push** con **CDU-17 PULL** — un nuovo endpoint REST paginato che il SIA chiama autonomamente quando viene registrato un nuovo endpoint. Sistema regionale = hub (centro stella), SIA = spoke che si tira giù i dati.

- Zero push dal sistema regionale → zero onere infrastrutturale
- Riusa interamente il modello di sicurezza già definito per [[wiki/concepts/sicurezza-cdu-15-16\|Sicurezza CDU-15-16 — Modello Autorizzazione per Ente]] (token via API Manager APIMBBONE; `codice_ente` dal Gateway)
- Paginazione cursor-based → scala su qualunque volume
- Idempotente: SIA può rieseguire pull senza side-effect
- BATCH-03 eliminato dal SRS → modello dati semplificato (`cons_t_notifica` non viene popolata in massa)
- **Blocco temporaneo delle acquisizioni durante il pull (Variante A, obbligatoria):** breve indisponibilità per il `sotto_tipo_consenso` in allineamento — accettabile perché gli allineamenti sono rari (onboarding nuovo endpoint)

---

## 2. Problema con design attuale (BATCH-03 push)

### Disegno SRS bozza v3 §7.3
```
Operatore CSI registra nuovo endpoint (CDU-14)
  → cons_r_asr_endpoint.stato_allineamento = DA_ALLINEARE
  → BATCH-03 triggerato async:
      ├─ legge TUTTI i consensi attivi della sotto-tipologia (cons_t_consenso)
      ├─ INSERT N record in cons_t_notifica (uno per consenso)
      ├─ BATCH-01 (ogni 5 min) consegna via SOAP al nuovo endpoint SIA
      └─ stato_allineamento → COMPLETATO
```

### Perché il cliente lo rifiuta

| Critica cliente | Causa tecnica |
|---|---|
| "ci carichiamo di onere non dovuto" | Sistema regionale paga: CPU lettura DB, storage `cons_t_notifica`, banda SOAP outbound, retry/error handling su milioni di record per ASR |
| "centro stella!!!" | Cliente vuole architettura hub-and-spoke: regionale espone dato, SIA pull. Allineamento è responsabilità del consumatore, non del produttore |
| Saturazione `cons_t_notifica` | Tabella usata anche da BATCH-01 ordinario: una mass-insert da allineamento può degradare le notifiche real-time degli altri ASR |
| Difficoltà recovery | Se BATCH-03 si interrompe a metà, ripartenza idempotente richiede logica custom (record già inseriti, error rows in `cons_t_batch_errori`, etc.) |

### Variante B nel commento cliente
> "Or export dei dati ma con interruzione del servizio online fino al caricamento"

Cliente accetterebbe export massivo con downtime — peggiore della pull on-demand ma comunque preferibile al push. Documentata in §7 sotto per completezza.

---

## 3. Proposta — CDU-17 Snapshot consensi per allineamento endpoint

### 3.1 Endpoint

```
GET /api/v1/consensi/snapshot
    ?codice_ente={ente}
    &codice_consenso={sotto_tipo}
    &cursor={base64-opaque}
    &page_size={1..5000, default 1000}
    &since={iso-8601-optional}
```

| Parametro | Tipo | Note |
|---|---|---|
| `codice_ente` | string, required | Codice ASR — vincolato dal `client_id` autenticato (vedi §4) |
| `codice_consenso` | string, required | Sotto-tipologia di consenso (es. `ROL`) — solo i consensi di quel tipo |
| `cursor` | string, optional | Opaque cursor per pagina successiva. Assente alla prima chiamata |
| `page_size` | int, optional | Dimensione pagina (1..5000, default 1000) |
| `since` | ISO-8601 datetime, optional | Restituisce solo consensi con `data_ultima_modifica > since`. Utile per resync incrementale |

### 3.2 Response 200

```json
{
  "codice_ente": "010",
  "codice_consenso": "ROL",
  "snapshot_taken_at": "2026-05-14T11:30:00Z",
  "page_size": 1000,
  "items": [
    {
      "codice_fiscale": "RSSMRA80A01L219X",
      "codice_consenso": "ROL",
      "codice_ente": "010",
      "stato_consenso": "ATTIVO",
      "valore_consenso": "SI",
      "data_espressione": "2024-09-12T14:20:11Z",
      "data_inizio_validita_consenso": "2024-09-12",
      "data_fine_validita_consenso": null,
      "informativa": { "id": 42, "versione": "3.1", "data_decorrenza": "2024-06-01", "data_scadenza": null }
    }
  ],
  "next_cursor": "eyJjb25zX2lkIjogMTIzNDU2fQ==",
  "has_more": true
}
```

Quando `has_more=false` e `next_cursor=null`, allineamento iniziale completato.

### 3.3 Pattern paginazione

- **Cursor opaco** (base64 JSON con `last_cons_id`) — niente offset (offset degrada O(n²) su PostgreSQL)
- Stabile rispetto a INSERT concorrenti: la query include `WHERE cons_id > :cursor` su PK, mai duplicati
- Stateless server-side — il cursor codifica tutto lo stato necessario
- Idempotente: stesso cursor → stesso risultato (se non arrivano UPDATE su record già paginati)

### 3.4 Confirmation endpoint (chiusura ciclo)

```
PATCH /api/v1/endpoints/{endp_id}/stato-allineamento
Body: { "stato": "COMPLETATO" }
```

SIA conferma fine allineamento. Sistema regionale aggiorna `cons_r_asr_endpoint.stato_allineamento` → `COMPLETATO` e sblocca CDU-03/CDU-09 se erano bloccati (vedi §5).

### 3.5 Notifica esito alla webapp (passo 5, nuovo — call 20/07/2026)

A chiusura, il SIA **comunica alla webapp Gestione Consensi** lo stato **`COMPLETATO`** e i **dati dell'ultimo invio andato a buon fine**, tramite la **stessa canalità di PULL-02** (email e/o webhook, configurabile). La webapp aggiorna lo stato visibile all'operatore Back Office.

### 3.6 Servizi endpoint esposti alle aziende (via API Manager)

I servizi di **inserimento / modifica / eliminazione endpoint** previsti dalla web app (famiglia CDU-14) sono **esposti anche alle aziende**, che li richiamano **passando dall'API Manager (APIMBBONE)**. Ciò consente ai Sistemi Informativi Aziendali di:
- **allineare i consensi** (CDU-17, questo documento);
- **bloccare l'acquisizione dei consensi durante la manutenzione** → scenario dedicato [[Manutenzione-endpoint_diagramma-sequenza\|Manutenzione endpoint]] (stato `IN_MANUTENZIONE`).

> ⚠️ **Doppia esposizione:** le operazioni effettuate **dalla web app** Gestione Consensi **NON** passano dall'API Manager (integrazione diretta). Solo le chiamate di **SIA/aziende** transitano dall'API Gateway APIMBBONE.

---

## 4. Sicurezza

**Identica a [[wiki/concepts/sicurezza-cdu-15-16\|Sicurezza CDU-15-16 — Modello Autorizzazione per Ente]]**, nessuna modifica:

- OAuth2 Client Credentials — token **emesso e validato dall'API Manager CSI (APIMBBONE)**, non da un Authorization Server a sé (aggiornamento 07/2026, cfr. [[wiki/concepts/sicurezza-cdu-15-16\|Sicurezza CDU-15/16]] §1.4)
- Il **Gateway APIM inoltra al backend il `codice_ente`** (con il CF da Shibboleth): l'isolamento per ente si basa su tale valore. **`cons_t_client_ente` è fuori scope V1.0** (estensione futura per multi-ente) — cfr. [[wiki/concepts/sicurezza-cdu-15-16\|Sicurezza CDU-15/16]] §3/§7
- Filter `EnteAuthorizationFilter` rigetta con 403 ogni mismatch (ente della request vs ente del Gateway)
- Repository query forza `WHERE codice_ente = :authorizedEnte` (ente preso dall'header del Gateway)

**Nuovo scope OAuth richiesto:**
- `consensi:snapshot` — distinto da `consensi:read` per consentire concessione granulare
- Motivo: snapshot è operazione "bulk" che può essere abilitata solo durante onboarding/allineamento, poi disattivata

**Rate limit dedicato:** quota separata per `/snapshot` (es. 600 req/min) configurata sull'**API Manager CSI (Traffic Manager APIM)** — non più `bucket4j` applicativo (07/2026) → impedisce che pull aggressivo di un SIA degradi le risposte CDU-15 real-time degli altri.

---

## 5. Workflow end-to-end

```
┌───────────────────────────────────────────────────────────────────┐
│  Operatore CSI                                                    │
│   1. CDU-14: registra nuovo endpoint                              │
│      → INSERT cons_t_endpoint                                     │
│      → INSERT cons_r_asr_endpoint (stato_allineamento=DA_ALLINEARE)│
└──────────────────────────────┬────────────────────────────────────┘
                               │
                               ▼
┌───────────────────────────────────────────────────────────────────┐
│  Notifica out-of-band al SIA                                      │
│   - Email al referente tecnico + webhook configurabile            │
│   - Payload: codice_ente, endp_id, codice_consenso, data_attivaz. │
│   - Sistema regionale NON deve garantire delivery — SIA è         │
│     responsabile di monitorare la propria casella                 │
└──────────────────────────────┬────────────────────────────────────┘
                               │
                               ▼
┌───────────────────────────────────────────────────────────────────┐
│  SIA chiama PATCH /endpoints/{id}/stato-allineamento              │
│     stato=IN_CORSO  (obbligatorio → blocca acquisizioni)          │
└──────────────────────────────┬────────────────────────────────────┘
                               │
                               ▼
┌───────────────────────────────────────────────────────────────────┐
│  SIA cicla:                                                       │
│   while next_cursor != null:                                      │
│      GET /api/v1/consensi/snapshot?codice_ente=...&cursor=...     │
│      processa items locamente                                     │
│   (nessun rate limit problem — cadenza controllata da SIA)        │
└──────────────────────────────┬────────────────────────────────────┘
                               │
                               ▼
┌───────────────────────────────────────────────────────────────────┐
│  SIA chiama PATCH /endpoints/{id}/stato-allineamento              │
│     stato=COMPLETATO                                              │
│   → cons_r_asr_endpoint.stato_allineamento = COMPLETATO           │
│   → CDU-03/09 sbloccano acquisizioni per quel sotto_tipo          │
└──────────────────────────────┬────────────────────────────────────┘
                               │
                               ▼
┌───────────────────────────────────────────────────────────────────┐
│  Passo 5 (nuovo) — SIA notifica la webapp Gestione Consensi       │
│   → stato COMPLETATO + dati ultimo invio andato a buon fine       │
│   → canalità PULL-02 (email e/o webhook)                          │
└───────────────────────────────────────────────────────────────────┘
```

---

## 6. Gestione race condition (consensi nuovi durante pull)

Durante il pull, possono arrivare nuovi consensi via CDU-03/09. Due varianti progettuali:

### Variante 6.A — Blocco acquisizioni (UNICA, obbligatoria)

- CDU-03/CDU-09 bloccano nuove acquisizioni per quel `sotto_tipo_consenso` finché `stato_allineamento=IN_CORSO`
- Pro: zero ambiguità, snapshot è punto-nel-tempo perfetto
- Contro: finestra di indisponibilità per cittadini (ma allineamenti sono rari, tipicamente onboarding nuovo endpoint)

> ✅ **Deciso (call CSI 20/07/2026 — PULL-01):** il blocco è **obbligatorio**. La Variante 6.B (watermark/no-blocco) è stata **ELIMINATA**.

### ~~Variante 6.B — Watermark + delta sync (no blocco)~~ — ELIMINATA (20/07/2026)

> ❌ **Rimossa su indicazione del committente.** Non si prevede più la modalità senza blocco con `X-Watermark-Cons-Id` / delta-sync. Conservata solo come riferimento storico.

---

## 7. Variante export file con downtime — NON PERSEGUITA (storico)

> ❌ **Non perseguita (call CSI 20/07/2026):** con il rifacimento CDU-17 il committente ha scelto la modalità **PULL paginata con blocco** (Variante A). L'opzione export-con-downtime (seconda preferenza del commento TR34 originale) **non viene realizzata**; conservata sotto solo come riferimento storico.

Per completezza, opzione coerente con la seconda preferenza nel commento TR34 ("export dei dati ma con interruzione del servizio online"):

| Aspetto | Descrizione |
|---|---|
| Formato | CSV/JSON.GZ generato da job on-demand |
| Storage | S3 compatible (CSI MinIO?) o SFTP fornito da CSI |
| Workflow | Operatore CSI dispatcha export → job genera file → URL pre-signed inviato a SIA → SIA scarica → conferma |
| Downtime | CDU-03/09 bloccati per intero periodo (generazione + download + load lato SIA) |
| Volume tipico | File singolo, anche multi-GB con compressione gzip |
| Sicurezza | URL pre-signed scaduto in 24h, IP allowlist SIA |

**Confronto con PULL CDU-17:**

| Criterio | CDU-17 PULL | Export file |
|---|---|---|
| Downtime cittadini | Solo durante IN_CORSO (variante A) o zero (variante B) | Intero ciclo export+download+load |
| Onere sistema regionale | Risposta REST on-demand | Job batch dedicato + storage temporaneo |
| Complessità SIA | Loop HTTP semplice | Download + parser file custom |
| Recovery | Idempotente, retry banale | Re-download intero file |
| Auditability | Per-request log strutturato | File singolo, audit grossolano |
| Integrazione con CDU-15/16 | Stesso pattern, zero nuovo concept | Canale parallelo, formato divergente |
| Volume scalabile | Sì (paginazione) | Limite pratico size file |

Raccomandazione: **CDU-17 PULL** come primaria. Export disponibile come fallback per ASR senza capacità integrazione REST (improbabile dato che CDU-15/16 già richiedono REST).

---

## 8. Confronto con design attuale BATCH-03

| Dimensione | BATCH-03 push (attuale SRS) | CDU-17 PULL (proposta) |
|---|---|---|
| Iniziativa | Sistema regionale | SIA |
| Direzione flusso | Outbound SOAP push | Inbound REST pull |
| Tabella `cons_t_notifica` | Popolata in massa (N record) | Non popolata |
| Onere infrastrutturale | Sistema regionale | SIA |
| Coerenza con CDU-15/16 | Diverso paradigma (SOAP push) | Stesso paradigma (REST pull) |
| Sicurezza | Da definire (X509 outbound) | Riusa OAuth2 CC + JWT esistente |
| Retry/recovery | Custom in BATCH-03 + `cons_t_batch_errori` | Idempotente per costruzione |
| Volume scalabile | Sì ma onere proporzionale | Sì, paginazione cursor |
| Downtime | Sì (CDU-03/09 bloccati durante IN_CORSO) | Sì, breve (blocco obbligatorio durante IN_CORSO — Variante A) |
| Modello dati | +1 tabella errori | Niente nuove tabelle |

---

## 9. Gap SRS da chiudere

| # | Sezione SRS | Modifica richiesta |
|---|---|---|
| G1 | §7.3 BATCH-03 | Marcare deprecato; sostituire con riferimento al nuovo §6.17 CDU-17. Oppure rimuovere completamente §7.3 |
| G2 | §6 Casi d'uso | Aggiungere nuovo §6.17 CDU-17 con testo proposto in §11 sotto |
| G3 | §6.14 CDU-14 | Modificare step 6: dopo INSERT endpoint, sistema invia notifica out-of-band a SIA (canale da definire). Eliminare trigger BATCH-03 |
| G4 | §8 Modello dati | Rimuovere `cons_t_batch_errori` se non più necessaria (verificare uso da BATCH-01/02) |
| G5 | §6.15/§6.16 | Aggiornare sezione "Modello di sicurezza" per includere scope `consensi:snapshot` (rimando a [[wiki/concepts/sicurezza-cdu-15-16\|Sicurezza CDU-15-16 — Modello Autorizzazione per Ente]]) |
| G6 | §10 Test plan | Aggiungere E2E: registrazione endpoint → notifica → pull paginato → conferma → consensi visibili lato SIA |
| G7 | §3.3 Componenti software | Aggiungere componente "Snapshot Service" (Spring Boot service con cursor pagination) |

---

## 10. Domande aperte CSI

| # | Domanda | Esito (call CSI 20/07/2026) |
|---|---|---|
| Q1 | ~~Variante 6.A (blocco) o 6.B (watermark)?~~ | ✅ **Variante A — blocco obbligatorio.** Variante B eliminata |
| Q2 | ~~Canale notifica out-of-band: email o webhook?~~ | ✅ **Entrambi**, via parametro di configurazione (solo email / solo webhook / entrambi). Webhook → SIA espone REST (contratto fornito da CSI) |
| Q3 | Scope OAuth `consensi:snapshot` + lifecycle | Delegato ad APIMBBONE (scope gestiti dall'API Manager). Resta lo swagger da produrre |
| Q4 | `page_size` massimo + tarabilità per ASR | ⚪ Da precisare (non bloccante) |
| Q5 | ~~Variante export-with-downtime~~ | ✅ **Non perseguita** — si adotta il PULL con blocco |
| Q6 | ~~Mantenere o rimuovere BATCH-03?~~ | ✅ **Rimosso** dal SRS (sostituito da §6.17 CDU-17) |
| Q7 | Conferma completamento via PATCH idempotente | PATCH idempotente confermato come modalità |
| Q8 | ~~SIA può fare chiamate REST attive? (PULL-08)~~ | ✅ **Sì** — i SIA/aziende chiamano i servizi via API Manager |

---

## 11. Testo proposto per inserimento in SRS

### 11.1 Nuovo §6.17 (sostituisce §7.3)

> **6.17 CDU-17: Snapshot consensi per allineamento endpoint (PULL)**
>
> **Attore primario:** Sistema Informativo Aziendale (SIA) della ASR a cui è stato associato un nuovo endpoint.
>
> **Obiettivo:** Permettere al SIA di acquisire autonomamente, in modalità PULL paginata, lo stato di tutti i consensi attivi della sotto-tipologia associata al nuovo endpoint, senza coinvolgere processi batch push dal sistema regionale.
>
> **Motivazione architetturale:** Modello hub-and-spoke ("centro stella") richiesto dal committente. Il sistema regionale espone il dato; il SIA, in qualità di consumatore, è responsabile dell'allineamento. Questo elimina l'onere infrastrutturale di push massivo dal sistema centrale e mantiene coerenza con il paradigma REST già adottato per CDU-15 e CDU-16.
>
> **Precondizioni:**
> - Un operatore con **profilo Back Office** (web app Gestione Consensi, accesso da PUA) ha registrato il nuovo endpoint via CDU-14 con `stato_allineamento = DA_ALLINEARE` — operazione **diretta, senza API Manager**
> - SIA autenticato tramite **API Manager APIMBBONE** con scope OAuth `consensi:snapshot`
> - Notifica out-of-band ricevuta dal SIA via **email e/o webhook** (PULL-02, configurabile)
>
> **Scenario principale:**
> 1. SIA chiama `PATCH /api/v1/endpoints/{endp_id}/stato-allineamento` con `stato=IN_CORSO`
> 2. Sistema imposta `cons_r_asr_endpoint.stato_allineamento = IN_CORSO` e **blocca (obbligatorio)** CDU-03/CDU-09 per quel `sotto_tipo_consenso` — snapshot coerente
> 3. SIA chiama in loop `GET /api/v1/consensi/snapshot?codice_ente={ente}&codice_consenso={cod}&cursor={c}&page_size={n}`
> 4. Sistema risponde con pagina di consensi attivi + `next_cursor` opaco (base64 di `cons_id` ultimo elemento); a ogni chiamata verifica che il SIA legga solo i dati del proprio ente (`codice_ente` dal Gateway)
> 5. Loop termina quando `has_more=false`
> 6. SIA chiama `PATCH /api/v1/endpoints/{endp_id}/stato-allineamento` con `stato=COMPLETATO`
> 7. Sistema imposta `stato_allineamento = COMPLETATO`, sblocca CDU-03/CDU-09
> 8. **(nuovo)** Il SIA **comunica alla webapp** lo stato `COMPLETATO` e i **dati dell'ultimo invio andato a buon fine**, tramite la canalità PULL-02
>
> **Servizi endpoint esposti alle aziende:** i servizi di inserimento/modifica/eliminazione endpoint (CDU-14) sono esposti anche alle aziende **via API Manager**; le operazioni della web app restano dirette (senza API Manager). Scenario di manutenzione (stato `IN_MANUTENZIONE`): vedi §7.4 SRS.
>
> **Specifiche tecniche API:** vedi swagger (OpenAPI) di CDU-17, da produrre.
>
> **Autenticazione:** token OAuth2 Client Credentials **emesso e validato dall'API Manager CSI (APIMBBONE)**, scope `consensi:snapshot`. Modello autorizzazione per ente identico a CDU-15/CDU-16 (`codice_ente` inoltrato dal Gateway; vedi §6.15/§6.16 "Modello di sicurezza").
>
> **Codici di risposta:** 200 OK, 400 (cursor invalido o parametri mancanti), 401 Unauthorized, 403 Forbidden (ente non autorizzato), 404 Not Found (endpoint non esistente), 409 Conflict (stato_allineamento già COMPLETATO), 429 Too Many Requests, 500 Internal Server Error.

### 11.2 Eliminazione §7.3

> **7.3 BATCH-03 (RIMOSSO)**
>
> Il processo BATCH-03 di allineamento massivo via push SOAP è stato rimosso dalla specifica TO-BE. L'allineamento di un nuovo endpoint con i consensi pre-esistenti è ora gestito in modalità PULL dal SIA tramite CDU-17 (vedi §6.17), in coerenza con il modello "centro stella" richiesto dal committente.

---

## 12. Riferimenti

- Decisione sicurezza: [[wiki/concepts/sicurezza-cdu-15-16\|Sicurezza CDU-15-16]] — AS-IS: Spring Security diretta (no API Gateway per fruitori esistenti); TO-BE: API Manager CSI Piemonte per nuovi fruitori (verbale 11/06/2026; vedi [[wiki/docs/adr/ADR-004-no-api-gateway\|ADR-004]])
- Pattern sicurezza riusato: [[wiki/concepts/sicurezza-cdu-15-16\|Sicurezza CDU-15-16 — Modello Autorizzazione per Ente]]
- Specifica OpenAPI fratelli CDU-15/16: [[wiki/analyses/analysis-2026-05-06-openapi-cdu-15-16\|analysis-2026-05-06-openapi-cdu-15-16]]
- Inventario consumer (SIA ASR): [[wiki/concepts/sistemi-esterni-integrati\|Sistemi Esterni Integrati]]
- Design batch corrente sotto revisione: [[wiki/concepts/batch-processes\|Processi Batch — BATCH-01, BATCH-02, BATCH-03]]
- Macchina a stati consensi: [[wiki/concepts/ciclo-vita-consenso\|Ciclo di Vita del Consenso]]

---

## ADR correlati

| ADR | Decisione |
|---|---|
| [ADR-006](ADR-006-batch-03-pull-cdu-17.md) | Sostituzione BATCH-03 push → CDU-17 PULL (questa concept è la fonte autoritativa) — **confermato dal committente 20/07/2026**, Variante B eliminata |
| [ADR-005](ADR-005-sicurezza-cdu-15-16.md) | Pattern sicurezza riusato |
| [ADR-007](ADR-007-batch-01-5min-skip-locked.md) | BATCH-01 5min (rimane attivo per notifiche puntuali) |
