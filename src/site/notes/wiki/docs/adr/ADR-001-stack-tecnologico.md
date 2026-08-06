---
{"dg-publish":true,"permalink":"/wiki/docs/adr/adr-001-stack-tecnologico/","title":"Stack tecnologico applicativo (Spring Boot 3 + Java 17 + Angular 19 + PostgreSQL 18)","tags":["stack","spring-boot","java","angular","postgresql"],"dg-note-properties":{"adr":1,"title":"Stack tecnologico applicativo (Spring Boot 3 + Java 17 + Angular 19 + PostgreSQL 18)","status":"accepted","date":"2026-03-02","updated":"2026-08-06","deciders":["CSI Piemonte","Exprivia"],"supersedes":[],"superseded-by":[],"tags":["stack","spring-boot","java","angular","postgresql"],"related_wiki":["[[wiki/concepts/stack-tecnologico-applicativo\|Stack Tecnologico Applicativo]]","[[wiki/sources/2026-03-12-pile-tecnologiche-csi\|Pile Tecnologiche CSI Piemonte]]","[[architettura-iaas|Architettura IaaS]]"],"sources":["[[wiki/sources/2026-03-02-domande-srs-csi-v02\|Domande SRS Consensi — Revisione CSI V02]] Q&A #2, #5, #10","[[2026-03-02-conspref-srs-v1-revised|CONSPREF-SRS-V1.0 revised bozza v2]] §3"]}}
---


# ADR-001: Stack tecnologico applicativo

## Status

`accepted` — confermato da CSI in Q&A V02 e formalizzato in SRS bozza v2 §3.

> 🔄 **Aggiornamento 06/08/2026 (call CSI):** versione database aggiornata **PostgreSQL 17 → 18** (CSI ha nel frattempo migrato il DBaaS). Resto dello stack invariato.

## Context

Rifacimento applicativo Gestione Consensi richiede uno stack moderno allineato alle pile tecnologiche correnti di CSI Piemonte. AS-IS gira su PostgreSQL 9 (RETIRED) con stack legacy non documentato. Vincoli:

- Pile CSI hanno classificazione CURRENT/DEPRECATED/RETIRED — solo CURRENT ammesso per nuovi progetti
- Stack target deve essere supportato dalla piattaforma [[wiki/concepts/architettura-iaas\|ECaaS Nivola]] (Helm chart + immagini Artifactory CSI)
- Java LTS richiesto dalla policy CSI

## Decision

Lo stack TO-BE è:

| Layer | Tecnologia | Versione |
|---|---|---|
| Frontend SPA | Angular | 19.x |
| Frontend UI library | QUASAR CSI | v2.1.0 |
| Web Server | Apache HTTPD (`httpd_csi` docker-base) | — |
| Backend | Spring Boot | 3.4.10+ |
| Linguaggio | Java | 17 LTS |
| Sicurezza | Spring Security | 6.x |
| Database | PostgreSQL | 18 (DBaaS Nivola) — era 17, aggiornato 06/08/2026 |

## Consequences

### Positive
- Conformità totale a Pile CSI CURRENT
- Spring Boot 3 abilita Jakarta EE 10, Java 17 LTS, native image (futuro)
- Angular 19 + QUASAR CSI library garantiscono riuso UI cross-applicativo sanità Piemonte
- PostgreSQL 18 abilita features sfruttate dal TO-BE (già disponibili da PG17): `SELECT FOR UPDATE SKIP LOCKED`, `gen_random_uuid()` nativa, MERGE statement

### Negative
- Migrazione PG9 → PG18 obbligatoria con salto di 9 major release (vedi [[wiki/docs/adr/ADR-013-migrazione-pg-dump-restore\|ADR-013-migrazione-pg-dump-restore]])
- Spring Boot 3 incompatible con Java 8/11 — niente backport possibile
- QUASAR CSI library richiede accesso repo CSI (dipendenza esterna)

### Neutral
- ~~Build/deploy gestiti da automation CSI (no skeleton ex-novo richiesto)~~ → Skeleton in carico a **Exprivia** (IaaS); confronto su POM con CSI (verbale 11/06/2026)

## Alternatives considered

| Alternativa | Motivo scarto |
|---|---|
| Spring Boot 2.7 + Java 11 | RETIRED da CSI; End-of-OSS-support raggiunto |
| React + Vite | Non in Pile CSI CURRENT; nessuna libreria component CSI |
| Quarkus | Non in Pile CSI; ECaaS standardizzato su Spring Boot |
| Node.js backend | Fuori scope policy CSI per progetti sanità |

## References

- [[wiki/sources/2026-03-02-domande-srs-csi-v02\|Domande SRS Consensi — Revisione CSI V02]] — Q&A #2 (Angular), #5 (Spring Boot 3.4.10+), #10 (PostgreSQL DBaaS, versione aggiornata a PG18 il 06/08/2026)
- [[wiki/sources/2026-03-12-pile-tecnologiche-csi\|Pile Tecnologiche CSI Piemonte]]
- [[wiki/concepts/stack-tecnologico-applicativo\|Stack Tecnologico Applicativo]]
- Correlato: [[wiki/docs/adr/ADR-002-piattaforma-ecaas\|ADR-002-piattaforma-ecaas]] piattaforma ECaaS, [[wiki/docs/adr/ADR-013-migrazione-pg-dump-restore\|ADR-013-migrazione-pg-dump-restore]] migrazione PG
