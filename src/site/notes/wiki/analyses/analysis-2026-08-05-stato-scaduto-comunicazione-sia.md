---
{"dg-publish":true,"permalink":"/wiki/analyses/analysis-2026-08-05-stato-scaduto-comunicazione-sia/","title":"Stato SCADUTO — semantica e comunicazione asincrona ai SIA/ASR (BAT-03)","tags":["scaduto","batch-02","sia","asr","semantica","asincrono","bat-03","integrazione","cdu-15","cdu-17"],"dg-note-properties":{"title":"Stato SCADUTO — semantica e comunicazione asincrona ai SIA/ASR (BAT-03)","aliases":["Stato SCADUTO — comunicazione asincrona SIA"],"type":"analysis","tags":["scaduto","batch-02","sia","asr","semantica","asincrono","bat-03","integrazione","cdu-15","cdu-17"],"created":"2026-08-05","updated":"2026-08-05","sources":["2026-03-02-conspref-srs-v1-revised","2019-03-20-acc-del-cdu-01-servizi-acquisizione","2019-06-01-webservice-consenso-regionale-v03"],"related":["[[analysis-2026-08-05-stato-scaduto-spiegato-semplice|Stato SCADUTO — Spiegato in Modo Semplice]]","[[ADR-016-scaduto-async-batch-02|ADR-016 SCADUTO async via BATCH-02]]","[[batch-processes|Processi Batch — BATCH-01, BATCH-02, BATCH-03]]","[[ciclo-vita-consenso|Ciclo di Vita del Consenso]]","[[analysis-gap-as-is-to-be|Analisi Gap AS-IS → TO-BE]]","[[alternativa-batch-03-pull|Alternativa BATCH-03 — PULL CDU-17]]","[[analysis-2026-05-14-punti-aperti-csi|Punti Aperti CSI]]","[[analysis-2026-05-06-openapi-cdu-15-16|OpenAPI CDU-15/16]]"]}}
---


# Stato SCADUTO — come funziona oggi e cosa manca per i SIA/ASR

**Scopo.** Documento di preparazione alla call dedicata CSI su **BAT-03** ("lo Stato SCADUTO è un componente da gestire"). Riporta lo stato consolidato ad oggi (wiki + SRS v1.0 revised v6), isola il buco reale e propone tre opzioni di comunicazione asincrona ai SIA.

**Stato del punto:** 🟠 aperto — call dedicata pending (deciso nella call CSI 20/07/2026, cfr. [[wiki/analyses/analysis-2026-05-14-punti-aperti-csi\|tracker]] riga BAT-03, Sprint 1).

---

## 1. In una riga

SCADUTO nel TO-BE è uno stato **corrente** del consenso, prodotto **solo** da BATCH-02 alla scadenza dell'informativa, che **non genera alcuna notifica verso le ASR** e **non compare** nello snapshot CDU-17: oggi un SIA può accorgersene **solo** interrogando CDU-15 caso per caso. Questo è il buco da chiudere.

---

## 2. Come funziona oggi — TO-BE

### 2.1 Chi produce lo stato

| Aspetto | Comportamento TO-BE | Riferimento |
|---|---|---|
| Chi imposta SCADUTO | **Esclusivamente BATCH-02** | SRS §5.1 nota SIA; §7.2; [[wiki/docs/adr/ADR-016-scaduto-async-batch-02\|ADR-016]] |
| Quando | Alla scadenza dell'informativa (`data_scadenza < NOW()`), con `annulla_consensi = NO` | SRS §7.2 ALG01/ALG02 |
| Frequenza | Giornaliera notturna (SRS §7.2); cadenza **non vincolante**, da concordare (call 20/07/2026) | [[wiki/concepts/batch-processes\|batch-processes]] §BATCH-02 |
| CDU-03 / CDU-04 | **Non** impostano più SCADUTO durante l'acquisizione | [[wiki/docs/adr/ADR-016-scaduto-async-batch-02\|ADR-016]] |

### 2.2 Cosa succede al dato

Alla scadenza dell'informativa **A** (SC67 risolto il 2026-07-23 — storicizzazione ancorata ad A):

1. `UPDATE` di chiusura del record corrente (`data_fine = NOW()`, `login_operazione = 'BATCH_SCADENZA_INF'`)
2. `INSERT` nuovo record con `tipo_stato = 'SCADUTO'`, `data_fine = NULL`, `d_informativa_id = A`, `valore_consenso` **copiato invariato**
3. `INSERT` in `cons_s_consenso` (storicizzazione, SRS §7.2)

Il `valore_consenso` non cambia: SRS §5.1 — *«Il valore espresso dal cittadino è ancora valido ma deve essere accettata la nuova informativa»*.

### 2.3 Chi viene avvisato

| Destinatario | SCADUTO | ANNULLATO |
|---|---|---|
| ASR / SIA | ❌ **nessuna notifica** | ✅ `cons_t_notifica` → BATCH-01 → SRV-04 SOAP |
| Cittadino | ❌ nessuna notifica | ✅ Notificatore Regionale UNP, direttamente da BATCH-02 (`flag_notifica_cittadino = TRUE`) |

Fonte: SRS §5.1 (*«Tale variazione non viene notificata alle aziende»*), §7.2; [[wiki/concepts/batch-processes\|batch-processes]] §BATCH-02; [[wiki/concepts/ciclo-vita-consenso\|ciclo-vita-consenso]] §Semantica degli stati.

### 2.4 Canali attraverso cui un SIA può vederlo

| Canale | Espone SCADUTO? | Note |
|---|---|---|
| **CDU-15** `GET /api/v1/consensi/stato` | ✅ sì — campo `stato_consenso` | Pull **puntuale**, un CF + un codice consenso alla volta. Enum e nota integratori già in `openapi-cdu-15-16-v0.1.yaml` |
| **CDU-17** `GET /api/v1/consensi/snapshot` | ❌ **no** | Lo snapshot esporta **solo i consensi attivi** (SRS §6.17); i record terminali SCADUTO/ANNULLATO non compaiono |
| **BATCH-01 / SRV-03** NotificaAcquisizioneConsenso | ❌ no | Innescato solo da acquisizione/variazione |
| **BATCH-01 / SRV-04** NotificaRevocaConsenso | ❌ no | Innescato solo da ANNULLATO |

**Conclusione:** nessun canale **push** e nessun canale **bulk/delta** copre SCADUTO. Solo polling puntuale CDU-15.

---

## 3. Differenza AS-IS → TO-BE — perché è un rischio

### 3.1 Il cambio di paradigma già documentato

| | AS-IS ([[wiki/sources/2019-03-20-acc-del-cdu-01-servizi-acquisizione\|ACC-DEL-CDU-01]]) | TO-BE |
|---|---|---|
| Chi imposta | Logica di acquisizione (servizio DA01/SRV-01) | BATCH-02 |
| Quando | Durante l'acquisizione, se esiste già un consenso per quel cittadino | Alla scadenza dell'informativa |
| Sincronia | Sincrono, dentro la transazione del servizio | Asincrono, latenza = schedulazione BATCH-02 |

### 3.2 ⚠️ Il punto più insidioso: **omonimia**, non solo asincronia

Nell'AS-IS `tipo_stato = 'scaduto'` marca il **record superato** durante la sovrascrittura logica:

```
UPDATE occorrenza: data_fine_validita = today, tipo_stato = 'scaduto'
INSERT nuovo consenso con nuovo valore
```

Cioè AS-IS: **scaduto = "questo record è vecchio, ignoralo"** — è sempre un record chiuso (`data_fine_validita` valorizzata).

Nel TO-BE: **SCADUTO = "questo è il record corrente, il valore vale ancora, ma l'informativa è da riaccettare"** — record aperto (`data_fine = NULL`).

> ⚠️ **Conseguenza:** un SIA che riusa la logica AS-IS scarta come "storico" un record che nel TO-BE è il consenso **vigente**. Non è una differenza di *timing*: è una **inversione di significato**. Va detto esplicitamente ai SIA, indipendentemente dal canale di comunicazione che verrà scelto.

Nota: nell'AS-IS lo stato "scaduto" **non transita mai sui webservice** verso le ASR — i tracciati SRV-01÷05 ([[wiki/sources/2019-06-01-webservice-consenso-regionale-v03\|WebService v03]]) non lo contengono. È sempre stato uno stato **interno**. Nel TO-BE diventa per la prima volta un valore **esposto** ai SIA via CDU-15.

---

## 4. Il buco da chiudere (BAT-03)

Tre fatti, messi insieme:

1. SCADUTO **non** genera notifica ASR (scelta SRS, coerente: il valore del consenso non cambia).
2. SCADUTO **non** compare nello snapshot CDU-17 (solo consensi attivi).
3. SCADUTO è **massivo e prevedibile**: alla scadenza di un'informativa transitano *tutti* i consensi collegati, in una sola finestra notturna.

Un SIA che mantiene una **copia locale** dello stato consensi (scenario realistico dopo l'allineamento CDU-17) **non ha modo di sapere** che quei consensi sono transitati in SCADUTO, se non ri-interrogando CDU-15 cittadino per cittadino.

**Domanda di fondo per la call:** SCADUTO deve essere un **evento notificabile** o resta un'informazione **a lettura on-demand**?

---

## 5. Opzioni per la comunicazione asincrona ai SIA/ASR

### Opzione A — Nessuna notifica, lettura real-time CDU-15 (baseline attuale)

Il SIA interroga CDU-15 **al momento dell'uso** del consenso (non tiene cache autoritativa).

- ✅ Zero sviluppo aggiuntivo, zero carico su BATCH-01/BATCH-02, coerente con SRS as-is
- ✅ Argomento sostanziale: in SCADUTO il **valore del consenso resta valido** → per l'operatività clinica del SIA **non cambia nulla**; cambia solo che il cittadino dovrà riaccettare l'informativa
- ❌ Inutilizzabile per SIA che tengono cache locale (proprio quelli che hanno fatto l'allineamento CDU-17)
- ❌ Non risolve l'omonimia AS-IS: serve comunque comunicazione documentale ai SIA

### Opzione B — Delta PULL su CDU-17 (raccomandata)

Estendere CDU-17 con un endpoint/parametro di **variazioni**:

```
GET /api/v1/consensi/variazioni?da=2026-08-04T00:00:00Z&stato=SCADUTO,ANNULLATO&cursor=...
```

- ✅ **Coerente con il modello "centro stella"** confermato dal committente il 20/07/2026: il regionale espone, il SIA pulla. Zero push, zero onere infra regionale
- ✅ Riusa security (OAuth2 CC + APIM + isolamento per ente), paginazione cursor-based e RFC 7807 già definiti per CDU-17
- ✅ Copre anche il caso "SIA fermo per manutenzione" senza code di retry
- ✅ Scala bene sul picco notturno: il SIA legge quando è pronto
- ❌ Richiede estensione OpenAPI CDU-17 + implementazione (stima: contenuta, riusa il pattern snapshot)
- ❌ Onere di scheduling in capo al SIA

### Opzione C — Push SOAP via BATCH-01, opt-in per ente

BATCH-02 inserisce in `cons_t_notifica` anche i record SCADUTO; BATCH-01 li invia via **SRV-03** con il campo `stato_consenso` nel tracciato; attivazione governata da un flag di configurazione **per ente/endpoint**.

- ✅ Nessun nuovo servizio: CSI ha confermato il 20/07/2026 che SRV-03/SRV-04 sono *«solo da svecchiare e integrare la nuova logica»*, e che **restano da definire i nomi esatti dei campi del tracciato** → l'aggiunta di `stato_consenso` rientra in quella finestra
- ✅ I SIA con cache locale restano allineati senza polling
- ❌ Contraddice la scelta SRS §5.1 ("non viene notificata alle aziende") → modifica documentale
- ❌ Picco di notifiche concentrato: la scadenza di un'informativa può generare decine di migliaia di record `cons_t_notifica` in una notte → dimensionamento BATCH-01 e retry
- ❌ Reintroduce onere push sul centro, già rigettato per BATCH-03

### Raccomandazione

**B come meccanismo, A come default per i SIA senza cache**, C solo se CSI impone il push.

Motivazione: B è l'unica opzione coerente con la decisione architetturale già presa e confermata dal committente ([[wiki/docs/adr/ADR-006-batch-03-pull-cdu-17\|ADR-006]], hub-and-spoke), riusa infrastruttura già progettata e non riapre il tema dell'onere infrastrutturale regionale che aveva affossato BATCH-03 push.

In tutti e tre gli scenari resta **obbligatoria** la comunicazione documentale sull'omonimia (§3.2).

---

## 6. Cosa portare alla call CSI (checklist BAT-03)

| # | Domanda | Perché serve |
|---|---|---|
| 1 | SCADUTO è evento **notificabile** o dato **a lettura on-demand**? | Decide fra A/B/C — tutto il resto discende |
| 2 | Quali SIA mantengono una **copia locale** dello stato consensi? | Se nessuno → opzione A sufficiente |
| 3 | Se notificabile: **push** (SRV-03 esteso) o **delta PULL** (CDU-17)? | Coerenza con il modello centro stella |
| 4 | Il tracciato SRV-03 può accogliere `stato_consenso`? | CSI ha lasciato aperti i nomi campi del tracciato |
| 5 | I SIA sono consapevoli che AS-IS `scaduto` ≠ TO-BE `SCADUTO`? (omonimia §3.2) | Rischio di scarto del record vigente |
| 6 | Serve un **preavviso** di scadenza informativa (N giorni prima) verso i SIA? | Permetterebbe di gestire il picco in anticipo |
| 7 | Cadenza BATCH-02 definitiva | Determina la latenza massima percepita dai SIA |
| 8 | Retention: per quanto un consenso resta consultabile in SCADUTO via CDU-15? | Impatta la logica di cache dei SIA |

---

## 7. Impatti documentali una volta chiusa la decisione

| Artefatto | Modifica attesa |
|---|---|
| SRS §5.1 | Ampliare la nota SIA con l'**omonimia** AS-IS/TO-BE (oggi cita solo l'asincronia) |
| SRS §7.2 | Se opzione C: aggiungere INSERT `cons_t_notifica` anche per SCADUTO + flag per ente |
| SRS §6.17 | Se opzione B: nuovo endpoint/parametro variazioni + criteri di delta |
| `openapi-cdu-15-16-v0.1.yaml` | Estendere la nota su `StatoConsenso` con l'omonimia; se B, aggiungere l'operazione delta |
| [[wiki/docs/adr/ADR-016-scaduto-async-batch-02\|ADR-016]] | Chiudere il punto aperto BAT-03 con la decisione presa |
| [[wiki/concepts/batch-processes\|batch-processes]] §Differenza semantica | Aggiornare da 🟠 aperto a ✅ risolto |
| [[wiki/analyses/analysis-2026-05-14-punti-aperti-csi\|Punti aperti]] | Chiudere riga BAT-03 |

---

## 8. Riferimenti puntuali

| Fonte | Punto |
|---|---|
| SRS v1.0 revised v6 §5.1 | Definizione stato "Scaduto" + nota SIA su asincronia |
| SRS v1.0 revised v6 §5.2 | Tabella stati e transizioni |
| SRS v1.0 revised v6 §6.15 | CDU-15, response con `stato_consenso` |
| SRS v1.0 revised v6 §6.17 | CDU-17 snapshot — **solo consensi attivi** |
| SRS v1.0 revised v6 §7.2 | BATCH-02 NotificaScadenzaInformative, ALG01/ALG02, gestione transazionale |
| [[wiki/docs/adr/ADR-016-scaduto-async-batch-02\|ADR-016]] | Decisione SCADUTO async, conseguenze, punti aperti |
| [[wiki/concepts/batch-processes\|batch-processes]] | §BATCH-02, §ALG02, §Differenza semantica AS-IS vs TO-BE |
| [[wiki/concepts/ciclo-vita-consenso\|ciclo-vita-consenso]] | Macchina a stati, tabella notifiche |
| [[wiki/analyses/analysis-gap-as-is-to-be\|Gap AS-IS → TO-BE]] | §Cambiamenti semantici critici #1 |
| [[wiki/sources/2019-03-20-acc-del-cdu-01-servizi-acquisizione\|ACC-DEL-CDU-01]] | Pseudo-codice AS-IS `tipo_stato = 'scaduto'` |
| `wiki/analyses/openapi-cdu-15-16-v0.1.yaml` | Enum `StatoConsenso` + nota integratori |
