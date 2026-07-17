---
{"dg-publish":true,"permalink":"/wiki/entities/csi-piemonte/","title":"CSI Piemonte","tags":["organizzazione","committente","infrastruttura","piemonte"],"dg-note-properties":{"title":"CSI Piemonte","aliases":["CSI Piemonte"],"type":"entity","tags":["organizzazione","committente","infrastruttura","piemonte"],"created":"2026-05-05","updated":"2026-07-16","sources":["2026-03-02-conspref-srs-v1-revised","2026-03-12-pile-tecnologiche-csi","2026-03-02-domande-srs-csi-v02"],"related":["[[Regione Piemonte]]","[[Architettura IaaS]]","[[wiki/sources/2026-03-12-pile-tecnologiche-csi\|Pile Tecnologiche CSI Piemonte]]","[[Gestione Consensi - Applicativo]]"]}}
---


# CSI Piemonte

Centro Servizi per il sistema informativo della Pubblica Amministrazione piemontese. Committente tecnico del progetto Gestione Consensi.

---

## Ruolo nel progetto

- Committente tecnico del rifacimento Gestione Consensi
- Gestore infrastruttura cloud IaaS/Nivola — provisioning in carico a CSI (07/2026: in questa fase solo ambienti **DEV** e **pre-prod**, PROD in fase successiva)
- Fornitore di componenti tecnici (QUASAR, CI/CD GitLab+Jenkins, Helm chart) — skeleton progetto in carico a Exprivia per CONSPREF (verbale 11/06/2026)
- Gestore sistemi integrati (GASP Salute, AURA, Gestione Deleghe, PUA)

---

## Sistemi CSI rilevanti

| Sistema                                    | Funzione                                        |
| ------------------------------------------ | ----------------------------------------------- |
| [[wiki/concepts/architettura-iaas\|Architettura IaaS]] | Infrastruttura IaaS Nivola (VM, deploy ADA/Chef) per hosting applicativi — non Kubernetes |
| Nivola                                     | Cloud IaaS/DBaaS                                |
| [[wiki/concepts/gasp-salute\|GASP Salute]]               | Identity Provider federato SPID/CIE             |
| AURA                                       | Anagrafe Unica Regionale Assistiti              |
| PUA                                        | Punto Unico di Accesso operatori                |
| IRIS                                       | Piattaforma gestione credenziali inter-servizio |
| Configuratore Regionale                    | Censimento applicazioni e profili operatori     |
| Notificatore UNP                           | User Notification Platform (email/push/IO/mex)  |

---

## [[wiki/sources/2026-03-12-pile-tecnologiche-csi\|Pile Tecnologiche CSI Piemonte]]

Stack approvato e vincolante per tutti i fornitori. Aggiornato periodicamente su kbt.csi.it.

---

## Contatti tecnici menzionati

- Referente anagrafiche: fornitore credenziali IRIS per AURA
- Referente GASP Salute: protocollo SAML2 confermato (verbale 11/06/2026); metadata SP ricevuti (TEST/preprod, 07/2026) — resta censimento federazione
- Referente CI/CD: architetture_ai@csi.it (per nuove immagini Docker)
- Referente Nivola: provisioning DBaaS (scheda formale)
