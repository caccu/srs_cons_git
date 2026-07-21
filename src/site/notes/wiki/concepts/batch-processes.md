---
{"dg-publish":true,"permalink":"/wiki/concepts/batch-processes/","title":"Processi Batch — BATCH-01, BATCH-02, BATCH-03","tags":["batch","notifica","scadenza","allineamento","soap","asincrono","rischio","mf64","mf66","mf33","skip-locked"],"dg-note-properties":{"title":"Processi Batch — BATCH-01, BATCH-02, BATCH-03","aliases":["Processi Batch — BATCH-01, BATCH-02, BATCH-03"],"type":"concept","tags":["batch","notifica","scadenza","allineamento","soap","asincrono","rischio","mf64","mf66","mf33","skip-locked"],"created":"2026-05-05","updated":"2026-07-20","sources":["2026-03-02-conspref-srs-v1-revised","2019-06-01-webservice-consenso-regionale-v03","2019-03-20-acc-del-cdu-01-servizi-acquisizione"],"related":["[[wiki/concepts/ciclo-vita-consenso\|Ciclo di Vita del Consenso]]","[[Gestione Consensi - Applicativo]]","[[wiki/sources/2019-06-01-webservice-consenso-regionale-v03\|Specifica WebService ConsensoRegionaleAziendale v03 (AS-IS)]]","[[Sistemi Esterni Integrati]]","[[valutazione-qualita-srs-consensi|Valutazione Qualità SRS — Gestione Consensi]]","[[wiki/concepts/alternativa-batch-03-pull\|Alternativa BATCH-03 — PULL CDU-17 (centro stella)]]","[[analysis-2026-05-14-risposte-mf-srs-v3]]"]}}
---


# Processi Batch

Processi asincroni del sistema TO-BE [[wiki/concepts/gestione-consensi-applicativo\|Gestione Consensi - Applicativo]]. Assenti nell'AS-IS — introdotti per disaccoppiare la logica di notifica/scadenza/allineamento dall'acquisizione sincrona.

> **Aggiornamento revisione SRS v3 (2026-05-14):**
> - BATCH-01: scheduling **5 minuti** con `SELECT FOR UPDATE SKIP LOCKED` (MF64R63) — sostituisce AS-IS 30 minuti
> - BATCH-02: SQL canonico per nuova informativa (MF66R65)
> - BATCH-03 push: **rimosso** — sostituito da PULL CDU-17 (MF69R68, ex TR34/TR68). Vedi [[wiki/concepts/alternativa-batch-03-pull\|Alternativa BATCH-03 — PULL CDU-17 (centro stella)]]
> - Notifica cittadino: via **Notificatore di Deleghe** (NON UNP), parte solo post-COMPLETATO (MF33R31)

---

## BATCH-01 — Notifica acquisizione consensi

**Trigger:** Ogni CDU che modifica un consenso inserisce record in `cons_t_notifica`
**Frequenza:** **Ogni 5 minuti** (MF64R63 — sostituisce AS-IS 30 min)
**Concorrenza:** `SELECT FOR UPDATE SKIP LOCKED` su `cons_t_notifica` per prevenire sovrapposizioni fra esecuzioni successive (MF64R63)
**Azione:** Chiama WebService SOAP verso ogni ASR coinvolta per notificare acquisizione/modifica consenso
**Contratto WSDL:** [[wiki/sources/2019-06-01-webservice-consenso-regionale-v03\|Specifica WebService ConsensoRegionaleAziendale v03 (AS-IS)]] — invariato nel TO-BE (confermato da [[wiki/entities/csi-piemonte\|CSI Piemonte]])

### Pattern di lock concorrenza (MF64R63)

```sql
-- Lettura batch di notifiche da inviare, con lock skip-locked per parallelismo sicuro
SELECT n.not_id, n.cf_cittadino, n.codice_consenso, n.endp_url, ...
FROM cons_t_notifica n
WHERE n.not_stato = 'DA_INVIARE'
  AND n.data_cancellazione IS NULL
ORDER BY n.data_creazione
LIMIT :batch_size
FOR UPDATE SKIP LOCKED;
```

Conseguenze:
- Più istanze (backend) di BATCH-01 possono girare in parallelo senza collisioni
- Notifiche non duplicate
- Notifiche in stato di lock da una run non sono prese da run successive — evitando sovrapposizione AS-IS che aveva motivato il 30 min

### Payload notifica

| Campo payload | Fonte |
|---|---|
| `codFiscale` | Consenso espresso |
| `codAsr` | Endpoint destinatario |
| `codConsenso`, `valConsenso` | Consenso espresso |
| `dataAcquisizione` | Timestamp acquisizione |
| `codOperatore`, `fonte` | Tracciatura origine CDU |

### Selezione endpoint (MF24R23)

Vedi 6.14 CDU-14 (Gestione ente ed endpoint) e ALG01 (Selezione nuovi endpoint) per la logica di selezione degli endpoint in base alla configurazione dell'ente.

### Tabella transizioni stato `cons_t_notifica` (MF64R63 — da aggiungere SRS)

| Da | A | Trigger |
|---|---|---|
| (creazione record) | `DA_INVIARE` | INSERT da CDU che acquisisce/modifica consenso |
| `DA_INVIARE` | `IN_INVIO` | BATCH-01 lock + invio in corso |
| `IN_INVIO` | `INVIATO` | Response 2xx da SIA |
| `IN_INVIO` | `ERRORE` | Response 4xx/5xx o timeout |
| `ERRORE` | `IN_INVIO` | Retry da BATCH-01 (con backoff) |
| `ERRORE` | `ERRORE_PERMANENTE` | Dopo 3 tentativi falliti (SRS §7.1 ALG02 / §8.4.2) → `cons_t_notifica_errore_dett` |

### Gestione errori

Retry con backoff (numero tentativi = [PROPOSTA] §7.1). Errori persistenti → `cons_t_notifica_errore_dett`.

### Notifica al cittadino post-COMPLETATO (MF33R31)

> Notifica cittadino/delegato via **Notificatore di Deleghe** parte **SOLO dopo conferma notifica alle aziende** (stato = COMPLETATO).

Implicazione: dipendenza temporale stretta. Quando tutti i record `cons_t_notifica` per una variazione raggiungono `INVIATO`/`COMPLETATO`, viene innescata la notifica al cittadino. Se BATCH-01 ritarda o fallisce, anche la notifica al cittadino è ritardata.

Distinzione canale:
- **Notificatore di Deleghe**: conferma post-acquisizione al cittadino/delegato (questo caso)
- **Notificatore UNP**: altre notifiche applicative generiche (vedi [[wiki/concepts/sistemi-esterni-integrati\|Sistemi Esterni Integrati]])

### ⚠️ RISCHIO CRITICO — Ambiguità operazione WSDL

> **Problema:** SRS §7.1 nomina "operazione Acquisizione" per BATCH-01. Nella [[wiki/sources/2019-06-01-webservice-consenso-regionale-v03\|Specifica WebService ConsensoRegionaleAziendale v03 (AS-IS)]], i servizi **uscenti** dal Regionale verso le ASR sono **SRV-03 NotificaAcquisizioneConsenso** — non SRV-01 AcquisizioneConsenso (che è inbound ASR→Regionale).
>
> Implementare BATCH-01 chiamando SRV-01 invece di SRV-03 = contratto WSDL sbagliato = errore grave di integrazione.
>
> Per le notifiche di **revoca/annullamento** l'operazione corretta è **SRV-04 NotificaRevocaConsenso** (anch'essa in uscita). L'SRS §7.1 recepisce SRV-03/SRV-04 come nota.
>
> ✅ **Confermato (call CSI 20/07/2026):** SRV-03/SRV-04 sono le operazioni esistenti, **"solo da svecchiare e integrare la nuova logica"**. Non ci sono nuovi servizi. **Resta solo** il dettaglio dei nomi esatti dei campi del tracciato, da recepire in implementazione.

---

## BATCH-02 — Scadenza informative

**Trigger:** Schedulato — **giornaliero notturno (SRS §7.2)**. ✅ **Call CSI 20/07/2026:** la cadenza esatta è **non vincolante**, da concordare in seguito (ipotesi confermata: una volta al giorno, notturno).
**Azione:** Quando un'informativa scade, aggiorna i consensi collegati in base al parametro `annulla_consensi`:

| `annulla_consensi` | Nuovo stato | Notifica ASR? |
|---|---|---|
| NO | SCADUTO | No |
| SI | ANNULLATO | Sì → INSERT `cons_t_notifica` → BATCH-01 |

> **Notifica al cittadino su ANNULLATO (SRS §7.2):** inviata **direttamente da BATCH-02** tramite **Notificatore Regionale (UNP)** con `flag_notifica_cittadino = TRUE` (distinta dalla conferma di rilascio, che va via Notificatore di Deleghe). Lo stato SCADUTO non genera notifica.

Vedi [[wiki/concepts/ciclo-vita-consenso\|Ciclo di Vita del Consenso]] per semantica completa degli stati.

### SQL canonico — determinazione nuova informativa (MF66R65)

Per ogni informativa scaduta, BATCH-02 deve determinare la **nuova informativa corrente** (quella ancora valida) per il medesimo `sotto_tipo_consenso`:

```sql
SELECT d_informativa_id AS nuova_d_informativa_id
FROM cons_d_informativa
WHERE sotto_tipo_consenso = :sotto_tipo_consenso_scaduta
  AND (data_scadenza IS NULL OR data_scadenza > NOW())
  AND data_decorrenza <= NOW()
  AND data_cancellazione IS NULL
ORDER BY data_decorrenza DESC
LIMIT 1;
```

Quindi seleziona tutti i consensi collegati all'informativa scaduta in stato `ATTIVO` o `NEGATO`:

```sql
SELECT cons_id, tipo_stato
FROM cons_t_consenso
WHERE d_informativa_id = :id_informativa_scaduta
  AND tipo_stato IN ('ATTIVO', 'NEGATO');
```

### ALG02 — Aggiornamento storicizzato (SC67 aperto)

> 🟠 **APERTO — SC67 (call CSI 20/07/2026: domanda da riformulare).** CSI **non ha compreso** la domanda posta in astratto. Va riproposta con uno scenario concreto che rende evidente la divergenza §6.13 vs §7.2:
>
> **Scenario.** L'informativa **A** (`annulla_consensi = NO`) scade e viene sostituita dall'informativa corrente **B** (`annulla_consensi = SI`), stesso `sotto_tipo_consenso`. Un consenso in stato **ATTIVO** collegato ad A viene storicizzato da BATCH-02.
>
> **Domanda mirata:** il nuovo stato del consenso deve essere **SCADUTO** (leggendo `annulla_consensi` dall'informativa **scaduta A**, come da §6.13) oppure **ANNULLATO** (leggendo il flag dalla **nuova B**, come implicherebbe l'SQL §7.2)? E qual è la **sorgente autoritativa** del flag (quale tabella/campo) al momento della storicizzazione?
>
> **Interpretazione tecnica proposta (da validare):** il flag descrive *cosa accade ai consensi alla scadenza di quella informativa* → sembra logico leggerlo dall'informativa **scaduta A** (§6.13). L'SQL §7.2 va allineato di conseguenza. La risposta CSI scioglie il nodo e uniforma il testo SRS.

Per ogni `cons_id` individuato:
1. **Chiudere il record corrente:** `UPDATE cons_t_consenso SET data_fine = NOW(), data_modifica = NOW(), login_operazione = 'BATCH_SCADENZA_INF' WHERE cons_id = :id AND data_fine IS NULL;`
2. **Inserire nuovo record storicizzato** con il nuovo stato (`SCADUTO` o `ANNULLATO`) puntando alla nuova informativa (`d_informativa_id = :nuova_d_informativa_id`).

Note tecniche (SRS §BATCH-02):
- `sotto_tipo_consenso` copiato direttamente dal record originale
- `d_informativa_id` → nuova informativa, non vecchia
- `endp_id = NULL` — consensi inseriti/aggiornati da batch non hanno endpoint origine
- `gen_random_uuid()` (PostgreSQL 17 nativa) per UUID nuovo record

### ⚠️ Differenza semantica AS-IS vs TO-BE — stato SCADUTO

> **AS-IS:** stato SCADUTO impostato durante l'operazione di acquisizione stessa, se il consenso è già presente ([[wiki/sources/2019-03-20-acc-del-cdu-01-servizi-acquisizione\|ACC-DEL-CDU-01 Servizi Acquisizione Consensi (AS-IS)]]).
>
> **TO-BE:** SCADUTO gestito **esclusivamente** da BATCH-02 alla scadenza dell'informativa.
>
> Paradigmi diversi. Se i [[wiki/concepts/sistemi-esterni-integrati\|Sistemi Esterni Integrati]] SIA ASR si aspettano comportamento AS-IS (scadenza sincrona durante acquisizione), il TO-BE potrebbe rompere l'integrazione.
>
> **Azione:** Documentare la differenza nella specifica OpenAPI CDU-15/16 e verificare con tutti i SIA coinvolti.
>
> 🟠 **Stato (call CSI 20/07/2026 — BAT-03):** CSI considera lo Stato SCADUTO un **componente da gestire** e ha rinviato a una **call dedicata** per definirne la gestione asincrona e la comunicazione alle ASR. Punto **ancora aperto**.

---

## Gestione Manutenzione ASR — Start/Stop Invio (verbale 11/06/2026)

Meccanismo concordato per gestire interruzioni programmate dei sistemi ASR, in particolare durante l'allineamento di nuovi dipartimentali aziendali.

### Logica di interruzione invio

| Scenario | Comportamento CSI |
|---|---|
| ASR segnala manutenzione programmata | CSI interrompe l'invio verso quel sistema |
| Un sistema di un'azienda non funziona | CSI interrompe l'invio verso **tutti i sistemi** della stessa azienda |
| Manutenzione completata | ASR segnala fine → CSI riprende l'invio |

### Servizi start/stop esposti dal sistema

Il sistema Gestione Consensi espone servizi che permettono alle ASR di segnalare:
- **Start manutenzione**: inizio periodo di interruzione programmata
- **Stop manutenzione**: fine periodo, sistema tornato operativo

**Onere in capo all'ASR**: è l'ASR che trasmette l'avviso di manutenzione via i servizi messi a disposizione — non è CSI a rilevare il problema proattivamente.

> ⚠️ **Gap aperto:** Questi servizi start/stop non sono ancora documentati nell'SRS né nella specifica OpenAPI. Da aggiungere come nuovi endpoint.

### Impatto su webapp cittadini

Quando un'azienda ASR è in manutenzione, la **web application cittadini** mostra messaggio esplicito di impossibilità a esprimere il consenso per quella specifica azienda.

### Allineamento nuovo dipartimentale

Alla nascita di un nuovo sistema dipartimentale aziendale, il processo di allineamento tra i dipartimentali aziendali **resta in capo all'azienda**. CSI interrompe l'invio fino al completamento.

---

## ~~BATCH-03~~ — sostituito da PULL CDU-17 (**proposed**, attende sign-off CSI)

> 🟡 **DESIGN RIVISTO — risposta MF69R68 (ex TR34/TR68):**
>
> BATCH-03 push viene **sostituito** dalla specifica TO-BE da modello **PULL** via CDU-17 (REST snapshot paginato, SIA pulla autonomamente). Proposta tecnica Exprivia (2026-05-14) come accoglimento del commento cliente TR68 "centro stella" — vedi [ADR-006](ADR-006-batch-03-pull-cdu-17.md), status `proposed`. Propagazione interna sulla wiki completata; **attende sign-off formale CSI Piemonte** prima di chiudere SRS e rimuovere fisicamente §7.3.
>
> Dettaglio progettuale completo: [[wiki/concepts/alternativa-batch-03-pull\|Alternativa BATCH-03 — PULL CDU-17 (centro stella)]]

Design storico (push, deprecato):
- Trigger: nuovo endpoint ASR (`cons_t_endpoint` con `stato_allineamento=DA_ALLINEARE`)
- Azione: invia consensi esistenti al nuovo endpoint via SOAP "allineamento massivo"

---

## Sequenza nel ciclo di vita del consenso

```
CDU-03 (rilascio) o CDU-04/05/10/11 (modifica)
  → INSERT cons_t_consenso
  → INSERT cons_t_notifica
  → [5 min, SKIP LOCKED] BATCH-01 → SRV-03 SOAP → SIA ASR
  → al completamento di tutte le notifiche aziende (stato=COMPLETATO)
     → Notificatore di Deleghe → notifica cittadino/delegato

Informativa scade
  → BATCH-02 → SELECT nuova informativa corrente (MF66)
              → UPDATE cons_t_consenso (chiude record)
              → INSERT cons_t_consenso (SCADUTO o ANNULLATO, nuova informativa)
              → [se ANNULLATO] INSERT cons_t_notifica → BATCH-01 → SIA ASR

Nuovo endpoint configurato (CDU-14 Back Office)
  → notifica out-of-band al SIA
  → SIA chiama CDU-17 GET /api/v1/consensi/snapshot (PULL paginato)
  → SIA conferma PATCH /endpoints/{id}/stato-allineamento → COMPLETATO
```

---

## Implicazioni tecniche

- Spring Boot 3: `@Scheduled` o Spring Batch per esecuzione periodica
- BATCH-01: client SOAP Apache CXF o Spring-WS
- BATCH-01 concorrenza: `SELECT FOR UPDATE SKIP LOCKED` (PostgreSQL 17 nativa) — più istanze pod sicure
- BATCH-02: usa `gen_random_uuid()` PostgreSQL 17 per UUID nuovi record storicizzati
- Certificati X509 WS-Security: richiederli a [[wiki/entities/csi-piemonte\|CSI Piemonte]] in Sprint 0
- `cons_t_batch_errori`: tabella TO-BE per tracciatura anomalie (non presente AS-IS)

---

## Punti aperti

| ID | Origine | Argomento | Stato |
|---|---|---|---|
| SC67 | Revisione SRS v3 lavorazione | Sorgente `annulla_consensi` (informativa scaduta vs nuova) in storicizzazione batch | 🟠 Aperto — **domanda riformulata a scenario** (call 20/07/2026: CSI non aveva capito) |
| BATCH-01 SRV-01 vs SRV-03 | Rischio interno | Ambiguità contratto WSDL outbound | ✅ Confermato (call 20/07/2026): SRV-03/SRV-04, "solo da svecchiare"; restano nomi campi |
| BATCH-02 frequenza | SRS non specifica | Schedulazione precisa BATCH-02 | ⚪ Non vincolante (call 20/07/2026), da concordare in seguito |

---

## ADR correlati

| ADR | Decisione |
|---|---|
| [ADR-007](ADR-007-batch-01-5min-skip-locked.md) | BATCH-01 5 min con SKIP LOCKED |
| [ADR-016](ADR-016-scaduto-async-batch-02.md) | Stato SCADUTO async via BATCH-02 |
| [ADR-006](ADR-006-batch-03-pull-cdu-17.md) | BATCH-03 push → CDU-17 PULL (**proposed**) |
| [ADR-012](ADR-012-notificatore-deleghe-post-completato.md) | Notifica cittadino via Notificatore di Deleghe post-COMPLETATO |
| [ADR-014](ADR-014-apache-cxf-soap-client.md) | Apache CXF client SOAP (BATCH-01 outbound) |
