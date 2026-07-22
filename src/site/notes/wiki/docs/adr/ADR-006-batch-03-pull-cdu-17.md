---
{"dg-publish":true,"permalink":"/wiki/docs/adr/adr-006-batch-03-pull-cdu-17/","title":"Sostituzione BATCH-03 push con CDU-17 PULL (centro stella)","tags":["batch-03","cdu-17","pull","centro-stella","allineamento","tr34","tr68","proposta"],"dg-note-properties":{"adr":6,"title":"Sostituzione BATCH-03 push con CDU-17 PULL (centro stella)","status":"accepted","date":"2026-07-20","deciders":["Marco Forneris","CSI Piemonte","Exprivia"],"supersedes":[],"superseded-by":[],"tags":["batch-03","cdu-17","pull","centro-stella","allineamento","tr34","tr68","proposta"],"related_wiki":["[[wiki/concepts/alternativa-batch-03-pull\|Alternativa BATCH-03 — PULL CDU-17 (centro stella)]]","[[wiki/concepts/batch-processes\|Processi Batch — BATCH-01, BATCH-02, BATCH-03]]","[[wiki/analyses/analysis-2026-05-14-tr34-alternativa-batch-03\|TR34 — Alternativa BATCH-03]]","[[sicurezza-cdu-15-16|Sicurezza CDU-15/16]]"],"sources":["[[analysis-2026-05-14-risposte-mf-srs-v3|Risposte MF — Revisione SRS v3 lavorazione]] MF69R68 (ex TR34/TR68)"]}}
---


# ADR-006: Sostituzione BATCH-03 push con CDU-17 PULL

## Status

`accepted` — proposta formalizzata da Exprivia il 2026-05-14 (risposta al commento **TR34/TR68**) e **confermata dal committente nella call CSI del 20/07/2026**, che ha fornito il **rifacimento completo** del caso d'uso.

> ✅ **Rielaborazione 20/07/2026 recepita:** **Variante B (watermark) eliminata** (blocco obbligatorio); **passo 5** (SIA notifica alla webapp `COMPLETATO` + dati ultimo invio, canalità PULL-02); **PULL-02** = email e/o webhook configurabile (webhook → SIA espone REST, contratto fornito da CSI); **servizi endpoint CRUD esposti alle aziende via API Manager**; **nuovo scenario di manutenzione** (stato `IN_MANUTENZIONE`); attore = operatore **Back Office** (web app da PUA); sicurezza allineata a 07/2026 (`codice_ente` dal Gateway, `cons_t_client_ente` fuori scope V1.0). Diagrammi: [[CDU-17_diagramma-sequenza\|CDU-17]] · [[Manutenzione-endpoint_diagramma-sequenza\|Manutenzione endpoint]].

## Context

Il design originale SRS bozza v3 §7.3 prevedeva **BATCH-03 push**: al provisioning di un nuovo endpoint SIA, il sistema regionale leggeva tutti i consensi attivi del sotto-tipo, popolava in massa `cons_t_notifica`, e BATCH-01 inviava SOAP outbound al SIA.

Il cliente ha rigettato l'approccio con commento TR34/TR68:
> "No, allineamento massivo deve avvenire con un passaggio di dati all'interno dell'azienda, altrimenti ci carichiamo di un onere che non ci è dovuto. Vogliamo spingere verso un centro stella!!!!! Or export dei dati ma con interruzione del servizio online fino al caricamento."

Critiche al push:
- Onere infrastrutturale sul sistema regionale (CPU, storage, banda)
- Saturazione `cons_t_notifica` può degradare notifiche real-time degli altri ASR
- Recovery custom complesso
- Paradigma divergente da CDU-15/16 (REST inbound)

## Decision

Rimuovere BATCH-03 push. Introdurre **CDU-17 — Snapshot consensi per allineamento endpoint** in modalità **PULL paginata**:

- Endpoint `GET /api/v1/consensi/snapshot?codice_ente={}&codice_consenso={}&cursor={}&page_size={}`
- Paginazione cursor-based opaca (base64 di `cons_id` ultimo elemento) — niente offset
- SIA pulla autonomamente alla cadenza preferita finché `has_more=false`
- Conferma completamento via `PATCH /api/v1/endpoints/{endp_id}/stato-allineamento`
- Sicurezza identica a [[wiki/docs/adr/ADR-005-sicurezza-cdu-15-16\|ADR-005-sicurezza-cdu-15-16]]: OAuth2 CC via **API Manager APIMBBONE** + scope `consensi:snapshot`; `codice_ente` inoltrato dal Gateway + `EnteAuthorizationFilter` + `WHERE codice_ente`. **`cons_t_client_ente` fuori scope V1.0** (estensione futura)
- Rate limit dedicato `/snapshot` gestito dall'**API Manager (Traffic Manager)** per non degradare CDU-15 real-time
- **Blocco obbligatorio** (ex Variante 6.A): CDU-03/CDU-09 bloccati durante `IN_CORSO`. **Variante 6.B (watermark) eliminata** (call 20/07/2026)
- **Passo 5:** il SIA notifica alla webapp `COMPLETATO` + dati ultimo invio, canalità PULL-02
- **Servizi endpoint CRUD** (CDU-14) esposti alle aziende via API Manager; web app diretta (no APIM)
- **Scenario manutenzione** (stato `IN_MANUTENZIONE`): blocco acquisizione + notifica webapp blocco/ripresa — [[Manutenzione-endpoint_diagramma-sequenza\|diagramma dedicato]]

## Consequences

### Positive
- Architettura hub-and-spoke ("centro stella") richiesta dal committente
- Zero push dal sistema regionale, zero onere infrastrutturale
- Zero popolamento massivo di `cons_t_notifica`
- Idempotente per costruzione (SIA può rieseguire pull)
- Riuso integrale modello sicurezza [[wiki/docs/adr/ADR-005-sicurezza-cdu-15-16\|ADR-005-sicurezza-cdu-15-16]] (nuovo scope `consensi:snapshot`)
- Coerenza paradigma REST con CDU-15/16
- BATCH-03 eliminato → modello dati semplificato (`cons_t_batch_errori` potenzialmente rimovibile)

### Negative
- Onere lato SIA (deve implementare client loop)
- Notifica out-of-band al SIA per trigger pull richiede canale separato (email + webhook configurabile)
- Variante A introduce indisponibilità CDU-03/CDU-09 durante `IN_CORSO`

### Neutral
- Nuovo CDU §6.17 nel SRS, rimozione §7.3
- Nuovo scope OAuth `consensi:snapshot`

## Alternatives considered

| Alternativa | Motivo scarto |
|---|---|
| **Mantenere BATCH-03 push** | Rigettato dal cliente nel commento TR34/TR68 |
| **Export file con downtime** (variante B cliente) | Coerente con seconda preferenza cliente ma peggiore su tutti i criteri: downtime intero ciclo, formato file divergente da REST, recovery = re-download intero |
| **gRPC streaming server-side** | Cliente CSI non ha pattern gRPC consolidato; nuovo stack di trasporto su 1 caso d'uso = non giustificato |

## Open issues

✅ **Chiusi nella call CSI 20/07/2026** (tracker [[wiki/analyses/analysis-2026-05-14-punti-aperti-csi\|Punti Aperti CSI]] §3):
- ~~PULL-01: blocco vs watermark~~ → **blocco obbligatorio** (Variante A); B eliminata
- ~~PULL-02: canale notifica~~ → **email e/o webhook** configurabile; webhook → SIA espone REST (contratto fornito da CSI)
- ~~PULL-05: export-with-downtime~~ → **non perseguita**
- ~~PULL-06: BATCH-03 rimozione~~ → **rimosso** dal SRS (§6.17 CDU-17)
- ~~PULL-08: SIA può chiamare REST~~ → **sì**
- PULL-03/PULL-04/PULL-07: scope lifecycle delegato ad APIMBBONE; `page_size` max e conferma idempotente da rifinire (non bloccanti)

**Residuo attivo:** produrre lo **swagger (OpenAPI) di CDU-17** per la sottoscrizione sull'APIM. Stato di manutenzione `IN_MANUTENZIONE`: modello dati/servizi start-stop da dettagliare (SRS §7.4).

## References

- [[wiki/concepts/alternativa-batch-03-pull\|Alternativa BATCH-03 — PULL CDU-17 (centro stella)]] (concept completo con scenario, sicurezza, gap SRS)
- [[wiki/analyses/analysis-2026-05-14-tr34-alternativa-batch-03\|TR34 — Alternativa BATCH-03]]
- [[wiki/concepts/batch-processes\|Processi Batch — BATCH-01, BATCH-02, BATCH-03]] §BATCH-03 RIMOSSO
- [[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|Risposte MF SRS v3]] MF69R68
- Correlato: [[wiki/docs/adr/ADR-005-sicurezza-cdu-15-16\|ADR-005-sicurezza-cdu-15-16]] sicurezza CDU-15/16 (pattern riusato), [[wiki/docs/adr/ADR-007-batch-01-5min-skip-locked\|ADR-007-batch-01-5min-skip-locked]] BATCH-01 5min
