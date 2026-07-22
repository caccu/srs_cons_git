---
{"dg-publish":true,"permalink":"/wiki/analyses/analysis-2026-05-14-tr34-alternativa-batch-03/","title":"TR34 — Alternativa BATCH-03 (centro stella PULL)","tags":["tr34","batch-03","cdu-17","pull","centro-stella","proposta","decisione-aperta","risposta-cliente"],"dg-note-properties":{"title":"TR34 — Alternativa BATCH-03 (centro stella PULL)","aliases":["TR34 — Alternativa BATCH-03 (centro stella PULL)"],"type":"analysis","tags":["tr34","batch-03","cdu-17","pull","centro-stella","proposta","decisione-aperta","risposta-cliente"],"created":"2026-05-14","updated":"2026-05-14","sources":["2026-03-02-conspref-srs-v1-revised"],"related":["[[wiki/concepts/alternativa-batch-03-pull\|Alternativa BATCH-03 — PULL CDU-17 (centro stella)]]","[[wiki/concepts/batch-processes\|Processi Batch — BATCH-01, BATCH-02, BATCH-03]]","[[sicurezza-cdu-15-16|Sicurezza CDU-15-16 — Modello Autorizzazione per Ente]]","[[wiki/analyses/analysis-2026-05-06-openapi-cdu-15-16\|analysis-2026-05-06-openapi-cdu-15-16]]","[[wiki/analyses/analysis-gap-as-is-to-be\|Analisi Gap AS-IS → TO-BE — Gestione Consensi]]"]}}
---


# TR34 — Alternativa BATCH-03 (centro stella PULL)

> **Mapping ID commento:** TR34 (CONSPREF-SRS-V1.0-revised_bozza_v3_CSI.pdf) = **TR68** (CONSPREF-SRS-V1.0-revised_bozza_v3_CSI_lavorazione.pdf, MF69R68). Stesso contenuto, numerazione cambiata. Riferimento unificato: [[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|analysis-2026-05-14-risposte-mf-srs-v3]].

**Quesito originale (CSI/Regione, commento TR34 / TR68 su CONSPREF-SRS-V1.0-revised_bozza_v3_CSI.pdf, righe 4951–4956, sez. §7.3):**

> "No, allineamento massivo deve avvenire con un passaggio di dati all'interno dell'azienda, altrimenti ci carichiamo di un onere che non ci è dovuto. Vogliamo spingere verso un centro stella!!!!! Or export dei dati ma con interruzione del servizio online fino al caricamento."

**Status decisionale:** ✅ **CONFERMATO dal committente (call CSI 20/07/2026)** — CDU-17 rielaborato per intero.

> ✅ **Aggiornamento 20/07/2026 — questo file è lo STORICO della proposta iniziale.** Il committente ha confermato CDU-17 fornendo il rifacimento completo. Rispetto a quanto scritto qui sotto: **Variante B (watermark) ELIMINATA** (blocco obbligatorio); aggiunto **passo 5** (notifica esito webapp); **PULL-02** = email e/o webhook configurabile (webhook → SIA espone REST); **servizi endpoint CRUD** esposti alle aziende via API Manager; **nuovo scenario di manutenzione** (`IN_MANUTENZIONE`); sicurezza aggiornata (`codice_ente` dal Gateway, **`cons_t_client_ente` fuori scope V1.0**). Fonte autoritativa corrente: [[wiki/concepts/alternativa-batch-03-pull\|Alternativa BATCH-03 — PULL CDU-17]] + diagrammi [[CDU-17_diagramma-sequenza\|CDU-17]] · [[Manutenzione-endpoint_diagramma-sequenza\|Manutenzione endpoint]].

**Data analisi:** 2026-05-14 (aggiornata 2026-07-20)

---

## Risposta sintetica al cliente

Proposta soluzione **semplice e indolore**: **CDU-17 PULL — Snapshot consensi per allineamento endpoint**.

- Sistema regionale espone endpoint REST paginato (`GET /api/v1/consensi/snapshot`)
- SIA pulla autonomamente, alla cadenza che preferisce, finché next_cursor è null
- Zero push, zero `cons_t_notifica` popolata in massa, zero downtime
- Riusa modello sicurezza CDU-15/16 (OAuth2 CC + JWT + tabella `cons_t_client_ente`)
- BATCH-03 eliminato dal SRS
- Variante B (export file con downtime) documentata come opzione retrocompatibile coerente con seconda preferenza cliente nel commento, ma sconsigliata: peggiore su tutti i criteri tranne semplicità lato SIA

**Dettaglio progettuale completo:** [[wiki/concepts/alternativa-batch-03-pull\|Alternativa BATCH-03 — PULL CDU-17 (centro stella)]]

---

## Punti chiave proposta

| # | Punto | Valore |
|---|---|---|
| 1 | Architettura | Hub-and-spoke ("centro stella" — rispetta esplicitamente terminologia cliente) |
| 2 | Endpoint | `GET /api/v1/consensi/snapshot` paginato cursor-based |
| 3 | Workflow | SIA: ack IN_CORSO → loop snapshot → ack COMPLETATO |
| 4 | Sicurezza | Riuso pattern CDU-15/16 + nuovo scope OAuth `consensi:snapshot` |
| 5 | Race condition | Variante A: blocco CDU-03/09 (default). Variante B: watermark no-block (opzionale) |
| 6 | Conferma | `PATCH /endpoints/{id}/stato-allineamento` idempotente |
| 7 | Notifica trigger | Out-of-band (email + webhook configurabile) — non garanzia delivery sistema regionale |
| 8 | Modello dati | Niente nuove tabelle; `cons_t_batch_errori` potenzialmente rimovibile |

---

## Decisione tracciata

| Aspetto | Stato | Owner |
|---|---|---|
| Proposta tecnica formalizzata | ✅ Fatto (2026-05-14) | Exprivia |
| Concept page wiki | ✅ [[wiki/concepts/alternativa-batch-03-pull\|Alternativa BATCH-03 — PULL CDU-17 (centro stella)]] | Exprivia |
| Testo proposto per SRS §6.17 | ✅ Pronto (vedi concept §11) | Exprivia |
| Validazione architetturale CSI | ⏳ In attesa | CSI Piemonte |
| Scelta variante 6.A vs 6.B (blocco vs watermark) | ⏳ Aperta | CSI Piemonte |
| Approvazione scope OAuth `consensi:snapshot` | ⏳ Aperta | CSI Piemonte |
| Aggiornamento SRS §6 e §7.3 | ⏳ Post-approvazione | Exprivia |
| Spec OpenAPI CDU-17 | ⏳ Post-approvazione | Exprivia |

---

## Impatti sulle altre pagine wiki

- [[wiki/concepts/batch-processes\|Processi Batch — BATCH-01, BATCH-02, BATCH-03]] — sezione BATCH-03 marcata "sotto revisione TR34"
- [[wiki/analyses/analysis-gap-as-is-to-be\|Analisi Gap AS-IS → TO-BE — Gestione Consensi]] — riga "Processi batch" da postillare (3 batch potenzialmente 2)
- [[wiki/analyses/analysis-2026-05-06-openapi-cdu-15-16\|analysis-2026-05-06-openapi-cdu-15-16]] — pattern paginazione cursor riutilizzabile come riferimento

---

## Cosa serve dal cliente

1. **OK architetturale** sulla scelta PULL vs export (preferenza cliente)
2. **Risposta a Q1–Q7** del concept page (variante race condition, canale notifica, scope OAuth, page_size, conferma idempotente, deprecation BATCH-03)
3. **Disponibilità Authorization Server CSI** per emissione scope `consensi:snapshot` (rimando aperto da TR30)

---

## Risposta da inserire come reply al commento TR34 (testo per PDF/Word)

> Testo pronto per essere incollato nel campo "rispondi" del commento TR34 del documento `CONSPREF-SRS-V1.0-revised_bozza_v3_CSI.pdf` o nella nuova revisione SRS. Tono formale, sintetico, action-oriented.

---

### Versione breve (comment reply, ~150 parole)

**Risposta Exprivia al commento TR34 (§7.3 BATCH-03)**

Accogliamo l'osservazione. Concordiamo che l'allineamento massivo non debba gravare sul sistema regionale e proponiamo una revisione architetturale conforme al modello "centro stella" indicato.

**Proposta:** sostituzione di BATCH-03 con un nuovo caso d'uso **CDU-17 — Snapshot consensi per allineamento endpoint**, in modalità **PULL** REST paginata. Il SIA, ricevuta notifica out-of-band di un nuovo endpoint registrato (CDU-14), invoca autonomamente l'endpoint `GET /api/v1/consensi/snapshot?codice_ente={ente}&codice_consenso={cod}&cursor={c}` finché esaurisce le pagine, quindi conferma il completamento via `PATCH /api/v1/endpoints/{id}/stato-allineamento`. Sicurezza identica a CDU-15/16 (OAuth2 Client Credentials + JWT).

**Vantaggi:** zero push, zero downtime, nessun popolamento massivo di `cons_t_notifica`, idempotenza nativa, cadenza controllata dal SIA, scalabilità per qualsiasi volume tramite paginazione cursor-based.

Il §7.3 viene rimosso; il nuovo §6.17 dettaglia CDU-17. Variante con export file e interruzione servizio disponibile come fallback.

Restiamo in attesa di conferma per procedere con la stesura della specifica OpenAPI CDU-17.

---

### Versione estesa (4 punti chiave, formato bullet)

**Risposta Exprivia al commento TR34 (§7.3 BATCH-03)**

1. **Accolta richiesta architetturale:** il sistema regionale non eseguirà più push massivo verso i SIA. BATCH-03 viene **rimosso** dalla specifica.

2. **Modello "centro stella" implementato via CDU-17 PULL:** nuovo endpoint REST `GET /api/v1/consensi/snapshot` paginato (cursor-based, opaco, stateless server-side). Il SIA pulla autonomamente i consensi attivi della sotto-tipologia associata al nuovo endpoint, alla cadenza che preferisce. Conferma completamento tramite `PATCH /api/v1/endpoints/{id}/stato-allineamento`.

3. **Riuso integrale del modello di sicurezza CDU-15/16** (OAuth2 Client Credentials + JWT firmato dall'AS CSI Piemonte + isolamento per ente via tabella `cons_t_client_ente`). Aggiunta del solo scope `consensi:snapshot` per concessione granulare, attivabile durante l'onboarding e revocabile a completamento.

4. **Benefici:** zero onere infrastrutturale sul sistema regionale, zero downtime del servizio online, zero popolamento massivo di `cons_t_notifica`, idempotenza nativa (SIA può rieseguire il pull senza side-effect), scalabilità trasparente su qualsiasi volume.

**Punti da definire congiuntamente con CSI:**
- Gestione finestra di allineamento: blocco delle acquisizioni CDU-03/CDU-09 per il `sotto_tipo_consenso` interessato (default, semplice) oppure modalità watermark senza blocco (più complessa, zero indisponibilità).
- Canale di notifica out-of-band al referente tecnico SIA all'attivazione del nuovo endpoint (email, webhook configurabile o entrambi).
- Politica scope OAuth `consensi:snapshot`: attivazione una tantum o permanente per SIA.

In allegato (oppure: nelle sezioni dedicate del SRS) viene fornita la bozza completa di §6.17 con scenario principale, codici di risposta, riferimenti di sicurezza e specifica OpenAPI in produzione.

La **variante "export con interruzione del servizio"** citata nel commento è progettualmente coerente con la seconda opzione indicata, ma sconsigliata rispetto al PULL: richiede downtime, formato file dedicato, gestione storage temporaneo e processo di download manuale. Resta documentata come fallback per ASR senza capacità di integrazione REST (situazione non prevista, dato che CDU-15 e CDU-16 già richiedono client REST lato SIA).

---

## ADR correlati

| ADR | Decisione |
|---|---|
| [ADR-006](ADR-006-batch-03-pull-cdu-17.md) | Sostituzione BATCH-03 push → CDU-17 PULL — **accepted** (confermato dal committente 20/07/2026; questa analysis è lo storico della proposta) |
| [ADR-005](ADR-005-sicurezza-cdu-15-16.md) | Pattern sicurezza riusato dal PULL |

