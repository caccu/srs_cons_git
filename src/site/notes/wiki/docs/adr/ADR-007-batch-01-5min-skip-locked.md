---
{"dg-publish":true,"permalink":"/wiki/docs/adr/adr-007-batch-01-5min-skip-locked/","title":"BATCH-01 scheduling 5 min con SELECT FOR UPDATE SKIP LOCKED","tags":["batch","batch-01","skip-locked","concorrenza","scheduling","mf64","postgresql"],"dg-note-properties":{"adr":7,"title":"BATCH-01 scheduling 5 min con SELECT FOR UPDATE SKIP LOCKED","status":"accepted","date":"2026-05-14","deciders":["Marco Forneris","Exprivia"],"supersedes":[],"superseded-by":[],"tags":["batch","batch-01","skip-locked","concorrenza","scheduling","mf64","postgresql"],"related_wiki":["[[wiki/concepts/batch-processes\|Processi Batch — BATCH-01, BATCH-02, BATCH-03]]","[[ciclo-vita-consenso|Ciclo di Vita del Consenso]]","[[Sistemi Esterni Integrati]]"],"sources":["[[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|Risposte MF SRS v3]] MF64R63","[[wiki/sources/2019-06-01-webservice-consenso-regionale-v03\|Specifica WebService ConsensoRegionaleAziendale v03 (AS-IS)]]"]}}
---


# ADR-007: BATCH-01 — scheduling 5 min con SKIP LOCKED

## Status

`accepted` — decisione MF64R63 sulla revisione SRS v3.

## Context

L'AS-IS prevedeva uno scheduling di **30 minuti** per la consegna delle notifiche di acquisizione consensi al SIA delle ASR. La motivazione storica era evitare sovrapposizione fra esecuzioni successive in caso di esecuzione lunga.

Limiti AS-IS:
- Latenza di notifica al SIA fino a 30 minuti → impatta SLA percepito
- Lock di tabella tradizionale rischia di bloccare run successive
- In ambiente multi-pod K8s ([[wiki/docs/adr/ADR-002-piattaforma-ecaas\|ADR-002-piattaforma-ecaas]]), più istanze BATCH-01 potrebbero girare in parallelo e generare duplicati o conflitti

PostgreSQL 18 ([[wiki/docs/adr/ADR-001-stack-tecnologico\|ADR-001-stack-tecnologico]], [[wiki/docs/adr/ADR-003-dbaas-nivola\|ADR-003-dbaas-nivola]]) supporta nativamente `SELECT FOR UPDATE SKIP LOCKED` — feature che consente lettura concorrente senza collisioni (disponibile fin da PG9.5).

## Decision

BATCH-01 viene riprogettato:

- **Scheduling: ogni 5 minuti** (sostituisce 30 min AS-IS)
- **Concorrenza:** `SELECT FOR UPDATE SKIP LOCKED` su `cons_t_notifica` per consentire esecuzione parallela sicura multi-pod
- Pattern SQL:

```sql
SELECT n.not_id, n.cf_cittadino, n.codice_consenso, n.endp_url, ...
FROM cons_t_notifica n
WHERE n.not_stato = 'DA_INVIARE'
  AND n.data_cancellazione IS NULL
ORDER BY n.data_creazione
LIMIT :batch_size
FOR UPDATE SKIP LOCKED;
```

- Macchina a stati `cons_t_notifica`: `DA_INVIARE` → `IN_INVIO` → `INVIATO` / `ERRORE` → `IN_INVIO` (retry) / `FALLITO_DEFINITIVO`
- Contratto WSDL outbound SOAP invariato (SRV-03 NotificaAcquisizioneConsenso, AS-IS)

## Consequences

### Positive
- Latenza notifica al SIA ridotta da 30 min → max 5 min
- Multi-pod safe nativo: nessuna sovrapposizione, niente duplicati
- Niente lock pessimistico di tabella — solo lock sui record presi
- Pattern dichiarativo, manutenibile

### Negative
- **Rischio aperto BAT-01:** ambiguità SRS §7.1 su quale operazione WSDL usare (SRV-01 inbound vs SRV-03 outbound) — vedi [[wiki/concepts/batch-processes\|Processi Batch]] §RISCHIO CRITICO. Da confermare con CSI prima implementazione
- Tuning `batch_size` necessario per bilanciare throughput vs lock contention
- Retry con backoff: parametri da definire (PROPOSTA SRS §7.1)

### Neutral
- Dipendenza forte da PostgreSQL ≥ 9.5 (SKIP LOCKED nativo, presente in PG18)

## Alternatives considered

| Alternativa | Motivo scarto |
|---|---|
| Mantenere 30 min + lock tabella | Latenza inaccettabile per SLA TO-BE; lock tabella problematico multi-pod |
| Cron singleton (HA leader election) | Complica deploy, aggiunge dipendenza esterna (etcd/ZK), niente parallelismo orizzontale |
| Queue esterna (RabbitMQ/Kafka) | Aggiunge nuovo middleware; vincoli ECaaS impediscono install Cluster-level |

## Open issues

- BAT-01: conferma scritta CSI su operazione WSDL (SRV-01 vs SRV-03) prima implementazione
- BATCH-02 scheduling preciso (vedi [[wiki/docs/adr/ADR-016-scaduto-async-batch-02\|ADR-016-scaduto-async-batch-02]])

## References

- [[wiki/concepts/batch-processes\|Processi Batch — BATCH-01, BATCH-02, BATCH-03]]
- [[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|Risposte MF SRS v3]] MF64R63
- [[wiki/sources/2019-06-01-webservice-consenso-regionale-v03\|Specifica WebService ConsensoRegionaleAziendale v03 (AS-IS)]]
- Correlato: [[wiki/docs/adr/ADR-001-stack-tecnologico\|ADR-001-stack-tecnologico]] stack (PG18 SKIP LOCKED nativo), [[wiki/docs/adr/ADR-006-batch-03-pull-cdu-17\|ADR-006-batch-03-pull-cdu-17]] CDU-17 PULL sostituisce BATCH-03, [[wiki/docs/adr/ADR-014-apache-cxf-soap-client\|ADR-014-apache-cxf-soap-client]] Apache CXF client SOAP, [[wiki/docs/adr/ADR-016-scaduto-async-batch-02\|ADR-016-scaduto-async-batch-02]] stato SCADUTO async
