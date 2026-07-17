---
{"dg-publish":true,"permalink":"/wiki/concepts/stack-tecnologico-applicativo/","title":"Stack Tecnologico Applicativo","tags":["stack","tecnologia","spring-boot","angular","postgresql","java"],"dg-note-properties":{"title":"Stack Tecnologico Applicativo","aliases":["Stack Tecnologico Applicativo","Stack Tecnologico","Stack Applicativo"],"type":"concept","tags":["stack","tecnologia","spring-boot","angular","postgresql","java"],"created":"2026-05-15","updated":"2026-07-16","sources":["2026-03-02-conspref-srs-v1-revised","2026-03-12-pile-tecnologiche-csi","2026-03-02-domande-srs-csi-v02"],"related":["[[Architettura IaaS]]","[[wiki/sources/2026-03-12-pile-tecnologiche-csi\|Pile Tecnologiche CSI Piemonte]]","[[Gestione Consensi - Applicativo]]","[[wiki/concepts/migrazione-postgres-9-17\|Migrazione PostgreSQL 9 → 17]]","[[wiki/concepts/sicurezza-cdu-15-16\|Sicurezza CDU-15/16]]"]}}
---


# Stack Tecnologico Applicativo

Stack TO-BE del progetto [[wiki/concepts/gestione-consensi-applicativo\|Gestione Consensi - Applicativo]]. Tutte le versioni allineate alle [[wiki/sources/2026-03-12-pile-tecnologiche-csi\|Pile Tecnologiche CSI]] CURRENT (marzo 2026).

---

## Riepilogo

| Layer | Tecnologia | Versione | Stato CSI |
|---|---|---|---|
| Frontend | Angular | 19.x | CURRENT |
| Frontend componenti | Componenti QUASAR (stack CSI SPA v2.1.0) | — | CURRENT |
| Web Server | Apache HTTPD (`httpd_csi` docker-base) | — | CURRENT |
| Backend | Spring Boot | 3.4.10+ | CURRENT |
| Linguaggio | Java | 17 LTS | CURRENT |
| Sicurezza | Spring Security | 6.x (Spring Boot 3 default) | CURRENT |
| Database | PostgreSQL | 17 (DBaaS Nivola) | CURRENT |
| Infrastruttura | [[wiki/concepts/architettura-iaas\|IaaS Nivola]] — provisioning CSI (07/2026: in questa fase solo **DEV** e **pre-prod**, PROD in fase successiva) | Nivola | CURRENT (verbale 11/06/2026, email CSI 07/2026) |

---

## Frontend — Angular 19

- **SPA Angular 19** servita da Apache `httpd_csi` (docker-base CSI)
- Componenti UI: componenti **QUASAR** forniti da CSI (stack SPA Angular2/SpringBoot/RESTEasy v2.1.0 — `v2.1.0` è la versione del pacchetto stack, non della libreria QUASAR). Accesso repository QUASAR da concordare con referente CSI prima avvio sviluppo UI.
- Pattern di rendering form: **Form Renderer dinamico** che consuma metadata backend (vedi [[wiki/concepts/composizione-dinamica-form-consenso\|Composizione Dinamica Form Consenso]])
- Build: `ng build --configuration=production`, copia in `dist/` → immagine Docker
- Routing: client-side, integrazione con autenticazione [[wiki/concepts/gasp-salute\|GASP Salute]] tramite redirect SPID/CIE

---

## Backend — Spring Boot 3.4.10+

- **Java 17 LTS** (richiesto da Spring Boot 3)
- **Spring Boot 3.4.10+** (range 3.4.10 ÷ 3.4.x come da Q&A CSI #5)
- **Spring Security**: gestione autenticazione/autorizzazione **applicativa** (no API Gateway esterno — vedi [[wiki/concepts/sicurezza-cdu-15-16\|Sicurezza CDU-15/16]])
- Persistenza: Spring Data JPA + driver PostgreSQL ufficiale (configurazione scram-sha-256 post-[[wiki/concepts/migrazione-postgres-9-17\|migrazione PG17]])
- Client SOAP: **Apache CXF** o **Spring-WS** (entrambi compatibili Spring Boot 3) per integrazione [[wiki/concepts/sistemi-esterni-integrati\|AURA, Gestione Deleghe, SIA-ASR]]
- Client REST: WebClient / RestTemplate per [[wiki/entities/notificatore-unp\|Notificatore UNP]]
- Scheduling: `@Scheduled` o Spring Batch per [[wiki/concepts/batch-processes\|processi batch]]
- OpenAPI: SpringDoc + openapi-generator (vedi `analysis-2026-05-06-openapi-cdu-15-16.md` §Integrazione Spring Boot)

### Due microservizi backend
1. **Servizio Gestione Consensi** — logica CDU (acquisizione, revoca, consultazione, storicizzazione)
2. **Servizio Configurazione** — parametri, informative, dizionari (`cons_d_*`)

---

## Database — PostgreSQL 17

- **DBaaS Nivola** ([[wiki/entities/csi-piemonte\|CSI Piemonte]]) — non PG nel pod Kubernetes
- Credenziali: gestite lato infrastruttura **IaaS CSI** (no K8s Secret — ambiente non Kubernetes)
- Funzionalità sfruttate: `SELECT FOR UPDATE SKIP LOCKED`, `gen_random_uuid()` nativa, MERGE
- Migrazione da PG9: vedi [[wiki/concepts/migrazione-postgres-9-17\|Migrazione PostgreSQL 9 → 17]] e [[wiki/analyses/conspref-dmp-tracker\|CONSPREF-DMP — Tracker Piano Migrazione Dati]]

---

## Infrastruttura — IaaS Nivola

> 🔄 **07/2026:** ambiente **IaaS Nivola**, deploy via **automation Chef (ADA)** + GitLab/Jenkins/SonarQube/Artifactory/ASGARD. I vincoli container qui sotto (Traefik/Cilium/Helm/GitOps/registry docker) derivano dal modello **ECaaS** delle *Linee Guida Cloud Native* e **non si applicano** all'ambiente IaaS; restano da precisare con CSI solo ingress/TLS e gestione segreti. Toolchain e URL: [[wiki/concepts/architettura-iaas\|Architettura IaaS]] §Toolchain CSI.

Vincoli storici ECaaS (da [[wiki/sources/2019-06-01-linea-guida-fornitori-cloud-native\|Linee Guida Cloud Native per Fornitori v1.0.1]] — non applicabili a IaaS):

| Componente | Nota |
|---|---|
| Ingress | TRAEFIK (ECaaS) — per IaaS: ingress/TLS da definire con CSI |
| CNI / Storage / Monitoraggio / Log | Cilium / NFS Trident / Prometheus / ELK (ECaaS) |
| **Deploy (IaaS)** | **ADA Deployer (cookbook Chef) su Nivola — no Helm/GitOps** |
| Registry | Artifactory CSI (repo/repart) |

Dettaglio: [[wiki/concepts/architettura-iaas\|Architettura IaaS]] §Toolchain CSI.

---

## Vincoli e linee guida

- **Immagini base obbligatorie:** `reference/spring-boot`, `angular`, `httpd_csi`, `maven` da `docker-base` Artifactory CSI *(vincolo ECaaS storico — da riverificare per IaaS)*
- **Multi-stage Dockerfile:** build Maven in stage build, JAR copiato in stage finale *(vincolo ECaaS storico — da riverificare per IaaS)*
- **Label OpenContainers obbligatorie** su ogni immagine *(vincolo ECaaS storico — da riverificare per IaaS)*
- ~~**Helm chart obbligatori:** `springboot` (backend) + `httpd` (web server) da `helm-base` CSI~~ — **superato (07/2026):** deploy IaaS via **ADA/Chef**, no Helm/GitOps (vedi [[wiki/concepts/architettura-iaas\|Architettura IaaS]] §Toolchain CSI)
- **Sicurezza API:** [[wiki/concepts/sicurezza-cdu-15-16\|AS-IS: Spring Security diretta (no API Gateway esterno); TO-BE: API Manager CSI Piemonte per nuovi fruitori esterni]] (verbale 11/06/2026)

---

## Pile RETIRED da evitare

| Tecnologia | Stato | Note |
|---|---|---|
| PostgreSQL 9 | RETIRED | Migrazione obbligatoria |
| Java 8 / 11 | RETIRED (per stack Boot 3) | Spring Boot 3 richiede Java 17+ |
| Spring Boot 2.x | RETIRED | End of OSS support |
| Angular ≤ 14 | RETIRED | — |

Vedi [[wiki/sources/2026-03-12-pile-tecnologiche-csi\|Pile Tecnologiche CSI Piemonte]] per elenco completo.

---

## ADR correlati

| ADR | Decisione |
|---|---|
| [ADR-001](ADR-001-stack-tecnologico.md) | Stack tecnologico (Spring Boot 3 + Java 17 + Angular 19 + PG17) |
| [ADR-002](ADR-002-piattaforma-ecaas.md) | Piattaforma ECaaS Kubernetes Nivola + vincoli |
| [ADR-014](ADR-014-apache-cxf-soap-client.md) | Apache CXF come client SOAP |
