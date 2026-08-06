---
{"dg-publish":true,"permalink":"/wiki/docs/adr/adr-003-dbaas-nivola/","title":"Database PostgreSQL su DBaaS Nivola, esterno all'infrastruttura IaaS","tags":["database","postgresql","dbaas","nivola","infrastruttura"],"dg-note-properties":{"adr":3,"title":"Database PostgreSQL su DBaaS Nivola, esterno all'infrastruttura IaaS","status":"accepted","date":"2026-03-02","deciders":["CSI Piemonte"],"supersedes":[],"superseded-by":[],"tags":["database","postgresql","dbaas","nivola","infrastruttura"],"related_wiki":["[[wiki/concepts/architettura-iaas\|Architettura IaaS]]","[[stack-tecnologico-applicativo|Stack Tecnologico Applicativo]]","[[migrazione-postgres-9-17|Migrazione PostgreSQL 9 → 17]]"],"sources":["[[wiki/sources/2026-03-02-domande-srs-csi-v02\|Domande SRS Consensi — Revisione CSI V02]] Q&A #10"]}}
---


# ADR-003: DBaaS Nivola, esterno all'infrastruttura IaaS

## Status

`accepted` — confermato da CSI in Q&A V02 #10. DB esterno all'infrastruttura IaaS Nivola.

## Context

Decisione su dove erogare PostgreSQL: dentro il pod K8s del namespace applicativo, oppure come servizio gestito esterno. Considerazioni:

- Persistenza dati critica per consensi sanitari (no perdita ammessa)
- Backup, patching, alta disponibilità richiesti da policy CSI
- Tenant isolation a livello DB per progetti sanità
- Capacità operativa: il team Exprivia non ha mandato di gestire DB

## Decision

PostgreSQL 18 (era 17, aggiornato 06/08/2026) viene erogato da **DBaaS Nivola** (servizio gestito CSI), esterno all'infrastruttura IaaS Nivola dell'applicativo.

- Provisioning via scheda formale a Nivola (alta latenza)
- Backup, patching, HA gestiti da Nivola
- Credenziali: gestite lato infrastruttura **IaaS CSI** → variabili env Spring; **mai nel codice sorgente** (no K8s Secret — ambiente non Kubernetes)
- HikariCP pool: `max-pool-size ≤ 40/replica` (istanza 100 conn max, 2 repliche)

## Consequences

### Positive
- Backup/HA/patching out-of-the-box (no carico operativo team applicativo)
- Tenant isolation gestita da Nivola
- Compliance e audit DB centralizzati

### Negative
- **Latenza provisioning alta** — bloccante per Sprint 1 se DEV non pronto giorno 1 (rischio R2 in [[wiki/analyses/analysis-2026-05-06-checklist-avvio-progetto\|Checklist Avvio Progetto]])
- DEV e PROD provisioning richiedono richieste formali separate
- Niente PG-extension custom installabili dall'app (vincolo Nivola)
- HikariCP pool size limitato dal max connessioni concesse da Nivola

### Neutral
- Migrazione PG9→PG18 deve passare per dump/restore (vedi [[wiki/docs/adr/ADR-013-migrazione-pg-dump-restore\|ADR-013-migrazione-pg-dump-restore]]) — niente accesso filesystem diretto

## Alternatives considered

| Alternativa | Motivo scarto |
|---|---|
| PostgreSQL in pod K8s | Persistenza problematica, no HA out-of-box, backup non garantiti |
| StatefulSet con PVC | Vincoli IaaS/Nivola impediscono storage non-NFS per dati critici |
| Cloud RDS esterno | Fuori policy CSI sanità (dati devono restare in Nivola) |

## References

- [[wiki/sources/2026-03-02-domande-srs-csi-v02\|Domande SRS Consensi — Revisione CSI V02]] Q&A #10
- [[wiki/concepts/architettura-iaas\|Architettura IaaS]] §DBaaS Nivola
- Correlato: [[wiki/docs/adr/ADR-002-piattaforma-ecaas\|ADR-002-piattaforma-ecaas]] piattaforma ECaaS, [[wiki/docs/adr/ADR-013-migrazione-pg-dump-restore\|ADR-013-migrazione-pg-dump-restore]] migrazione, [[wiki/analyses/conspref-dmp-tracker\|CONSPREF-DMP — Tracker Piano Migrazione Dati]]
