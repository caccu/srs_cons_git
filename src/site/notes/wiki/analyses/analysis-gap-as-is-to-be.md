---
{"dg-publish":true,"permalink":"/wiki/analyses/analysis-gap-as-is-to-be/","title":"Analisi Gap AS-IS → TO-BE — Gestione Consensi","tags":["gap-analysis","as-is","to-be","migrazione","delta","dismissione"],"dg-note-properties":{"title":"Analisi Gap AS-IS → TO-BE — Gestione Consensi","aliases":["Analisi Gap AS-IS → TO-BE — Gestione Consensi"],"type":"analysis","tags":["gap-analysis","as-is","to-be","migrazione","delta","dismissione"],"created":"2026-05-05","updated":"2026-06-17","sources":["2019-02-01-sfu-gestione-consensi-v1-7","2019-03-20-acc-del-cdu-01-servizi-acquisizione","2019-04-08-dizionario-dati-as-is","2019-06-01-webservice-consenso-regionale-v03","2026-03-02-conspref-srs-v1-revised","2023-09-01-conspref-srs-01-v03"],"related":["[[Gestione Consensi - Applicativo]]","[[wiki/concepts/ciclo-vita-consenso\|Ciclo di Vita del Consenso]]","[[batch-processes|Processi Batch — BATCH-01, BATCH-02, BATCH-03]]","[[Sistemi Esterni Integrati]]","[[wiki/sources/2019-04-08-dizionario-dati-as-is\|Modello Dizionario Dati AS-IS (2019)]]","[[wiki/sources/2026-03-02-conspref-srs-v1-revised\|CONSPREF-SRS-V1.0 revised bozza v2]]","[[wiki/sources/2019-02-01-sfu-gestione-consensi-v1-7\|Studio Funzionale Gestione Consensi SoL V1.7 (AS-IS 2019)]]","[[alternativa-batch-03-pull|Alternativa BATCH-03 — PULL CDU-17 (centro stella)]]","[[composizione-dinamica-form-consenso|Composizione Dinamica Form Consenso — Single Source of Truth]]","[[analysis-2026-05-14-risposte-mf-srs-v3]]"]}}
---


# Analisi Gap AS-IS → TO-BE — Gestione Consensi

**Scopo:** Identificare cosa cambia, cosa rimane invariato e cosa viene dismesso tra sistema AS-IS (2019) e TO-BE (SRS 2026). Input per sprint planning e gestione rischi integrazione.

**Fonti AS-IS:** [[wiki/sources/2019-02-01-sfu-gestione-consensi-v1-7\|Studio Funzionale Gestione Consensi SoL V1.7 (AS-IS 2019)]], [[wiki/sources/2019-03-20-acc-del-cdu-01-servizi-acquisizione\|ACC-DEL-CDU-01 Servizi Acquisizione Consensi (AS-IS)]], [[wiki/sources/2019-04-08-dizionario-dati-as-is\|Modello Dizionario Dati AS-IS (2019)]], [[wiki/sources/2019-06-01-webservice-consenso-regionale-v03\|Specifica WebService ConsensoRegionaleAziendale v03 (AS-IS)]]
**Fonti TO-BE:** [[wiki/sources/2026-03-02-conspref-srs-v1-revised\|CONSPREF-SRS-V1.0 revised bozza v2]], [[wiki/sources/2023-09-01-conspref-srs-01-v03\|CONSPREF-SRS-01-V03 Requisiti Gestione Consensi]]

---

## Sintesi delta

| Dimensione                       | AS-IS                         | TO-BE                                                                                                                                          | Tipo cambiamento                                                                |
| -------------------------------- | ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| Casi d'uso                       | 6 CDU                         | 16 CDU                                                                                                                                         | espansione funzionale                                                           |
| Modello dati                     | 12 entità                     | 25+ entità                                                                                                                                     | nuove entità batch/endpoint/allegati                                            |
| Stack DB                         | PostgreSQL 9 (RETIRED)        | PostgreSQL 17 (CURRENT)                                                                                                                        | Migrazione obbligatoria                                                         |
| Stack applicativo                | Legacy non documentato        | Spring Boot 3 + Angular 19                                                                                                                     | Rifacimento completo                                                            |
| Processi batch                   | Assenti                       | 2 batch attivi (BATCH-01/02) + BATCH-03 push **sostituito** da PULL CDU-17 ([[wiki/concepts/alternativa-batch-03-pull\|Alternativa BATCH-03 — PULL CDU-17]] / [ADR-006] — **accepted**, confermato dal committente 20/07/2026) | Nuovi — vedi [[wiki/concepts/batch-processes\|Processi Batch — BATCH-01, BATCH-02, BATCH-03]] |
| API esterne per SIA              | Solo SOAP inbound (SRV-01/02) | SOAP outbound + REST CDU-15/16                                                                                                                 | Espansione bidirezionale                                                        |
| Autenticazione Cittadino         | Non documentata               | [[wiki/concepts/gasp-salute\|GASP Salute]] SPID/CIE                                                                                                          | Modernizzazione                                                                 |
| Notifiche ASR                    | Sincrono durante acquisizione | Asincrono BATCH-01 ogni **5 min** con `SELECT FOR UPDATE SKIP LOCKED` (MF64R63 — sostituisce AS-IS 30 min)                                     | Cambio paradigma + ottimizzazione concorrenza                                   |
| Notifica cittadino               | Non documentata               | Via **Notificatore di Deleghe** (NON UNP), post-COMPLETATO aziende (MF33R31)                                                                   | Nuovo canale dedicato                                                           |
| Ricerca assistito operatore      | Non documentata               | AURA: FindProfiliAnagrafici + getProfiloSanitario; **NO SistemaTS** (MF53/MF55)                                                                | Stack semplificato                                                              |
| UI terminologia consenso         | Non standard                  | "acconsento"/"nego" UI, mapping tecnico ATTIVO/NEGATO invariato (MF9R8)                                                                        | Allineamento UX                                                                 |
| CDU-01                           | Singolo caso d'uso            | **Split CDU-01a Operatore** (RUPAR/IRIDE/SPID + profilazione) + **CDU-01b Cittadino** (SPID/CIE webapp separata) (MF16R15)                     | Separazione canali autenticazione                                               |
| CDU-04/CDU-05 lato cittadino     | Casi d'uso distinti           | **Inglobati**: webapp citt usa pulsante unico "Salva", CDU-05 separato solo per Operatore (MF45R44)                                            | Semplificazione UX                                                              |
| Canali acquisizione consenso     | webapp + operatore            | webapp + operatore + **LIS** (MF3/MF4)                                                                                                         | +1 canale (LIS)                                                                 |
| Form rendering Citt vs Operatore | Non vincolato                 | Form Renderer unico SSoT (MF57R56) — niente divergenze fra le 2 webapp                                                                         | Nuovo vincolo architetturale                                                    |
| Storicizzazione consensi         | `cons_s_consenso` non usata   | Usata attivamente                                                                                                                              | Attivazione feature esistente                                                   |
| Infrastruttura                   | Legacy                        | **IaaS Nivola** (DEV/TEST/PROD) — provisioning CSI (verbale 11/06/2026; SRS §3.5 ECaaS/Kubernetes superato)                                  | Migrazione cloud                                                                |

---

## CDU: cosa cambia

### CDU con continuità AS-IS → TO-BE

| Funzione AS-IS            | CDU TO-BE                              | Variazioni principali                                                               |
| ------------------------- | -------------------------------------- | ----------------------------------------------------------------------------------- |
| Acquisizione consenso     | CDU-03 Rilascio                        | Aggiunta visualizzazione informativa PDF obbligatoria + checkbox presa visione      |
| Modifica consenso         | CDU-04 Modifica con scadenza           | Aggiunta logica riaccettazione se SCADUTO/ANNULLATO                                 |
| Cambio valore             | CDU-05 Cambio valore                   | Stessa logica, no riaccettazione (informativa invariata)                            |
| Consultazione stato       | CDU-02 Consultazione                   | Redesign UI — cruscotto con card per tipo/sottotipo                                 |
| Acquisizione da operatore | CDU-07/08 Ricerca + gestione per conto | Separati, con ricerca [[wiki/concepts/sistemi-esterni-integrati\|Sistemi Esterni Integrati]] AURA |

### CDU nuovi nel TO-BE

| CDU       | Descrizione                             | Note                                             |
| --------- | --------------------------------------- | ------------------------------------------------ |
| CDU-01    | Autenticazione cittadino                | [[wiki/concepts/gasp-salute\|GASP Salute]] — blocco rischio #1 |
| CDU-06    | Download PDF attestazione consenso      | Nuova funzionalità — [PROPOSTA]                  |
| CDU-09/10/11 | Gestione consenso per conto assistito (op.) — rilascio, modifica, cambio valore | Nuovi CDU operatore; CDU-09/10/11 = equivalenti operatore di CDU-03/04/05 |
| CDU-12/13 | Back office tipi consenso e informative | Parametrizzazione dinamica                       |
| CDU-14    | Back office enti ed endpoint            | Gestione endpoint ASR — trigger BATCH-03         |
| CDU-15    | API stato consensi per SIA              | REST — [[wiki/analyses/analysis-2026-05-06-openapi-cdu-15-16\|v0.1-DRAFT prodotta]], 5 TBD CSI aperti |
| CDU-16    | API configurazione per SIA              | REST — [[wiki/analyses/analysis-2026-05-06-openapi-cdu-15-16\|v0.1-DRAFT prodotta]], 5 TBD CSI aperti |

---

## Modello dati: entità nuove TO-BE

Rispetto alle 12 entità AS-IS documentate in [[wiki/sources/2019-04-08-dizionario-dati-as-is\|Modello Dizionario Dati AS-IS (2019)]]:

| Categoria | Entità TO-BE nuove |
|---|---|
| Notifiche batch | `cons_t_notifica`, `cons_t_notifica_errore_dett`, `cons_t_batch_errori` |
| Endpoint ASR | `cons_t_endpoint`, `cons_r_asr_endpoint` |
| Allegati informativa | `cons_t_allegato`, `cons_d_allegato_tipo` |
| Parametri dinamici | `cons_d_parametro`, `cons_r_consenso_parametro` |
| Tracciatura esterna | `cons_t_traccia_serv_est` |
| Relazioni n:m nuove | `cons_r_sotto_tipo_cons_asr_endpoint`, `cons_r_consenso_valore`, `cons_r_informativa_asr` |

**Entità attivata (esistente ma non usata AS-IS):** `cons_s_consenso` — storico pre-modifica, usata attivamente nel [[wiki/concepts/ciclo-vita-consenso\|Ciclo di Vita del Consenso]] TO-BE per garantire no-sovrascrittura. Struttura da verificare nell'audit DDL Sprint 0.

---

## Integrazioni: delta dettagliato

| Sistema                      | AS-IS                                  | TO-BE                                                                        | Cambiamento                                     |
| ---------------------------- | -------------------------------------- | ---------------------------------------------------------------------------- | ----------------------------------------------- |
| SIA ASR (outbound)           | Sincrono durante acquisizione (SRV-01) | Asincrono [[wiki/concepts/batch-processes\|Processi Batch — BATCH-01, BATCH-02, BATCH-03]] | Paradigma asincrono — rischio semantica SCADUTO |
| SIA ASR (inbound)            | Non presente                           | CDU-15/16 REST OpenAPI                                                       | Nuovo canale — spec da produrre                 |
| AURA                         | Non documentata                        | SOAP + WS-Security IRIS                                                      | Nuovo (WSDL da richiedere)                      |
| Gestione Deleghe             | Non documentata                        | SOAP + OAuth2 Client Credentials                                             | Nuovo (WSDL da richiedere)                      |
| Notificatore UNP             | Non presente                           | REST                                                                         | Nuovo                                           |
| [[wiki/concepts/gasp-salute\|GASP Salute]] | Non documentata                        | OIDC/SAML2 (da definire)                                                     | Nuovo — rischio critico                         |

---

## Cambiamenti semantici critici (rischi integrazione)

### 1. Stato SCADUTO — meccanismo diverso

| | AS-IS ([[wiki/sources/2019-03-20-acc-del-cdu-01-servizi-acquisizione\|ACC-DEL-CDU-01 Servizi Acquisizione Consensi (AS-IS)]]) | TO-BE |
|---|---|---|
| Chi imposta SCADUTO | Logica acquisizione (SRV-01) | BATCH-02 su scadenza informativa |
| Quando | Durante acquisizione se consenso già presente | Alla scadenza dell'informativa |
| Notifica ASR | Sincrona | Asincrona (dopo BATCH-01, max 5 min dopo BATCH-02) |

> ⚠️ SIA ASR che si aspetta comportamento AS-IS riceve notifica in momento diverso. Documentare nella specifica OpenAPI CDU-15/16 e verificare con tutti i SIA coinvolti prima del go-live.

### 2. Storicizzazione — attivazione feature

`cons_s_consenso` esiste nel DB AS-IS ma non popolata. Nel TO-BE usata attivamente. Rischio: struttura fisica potrebbe non corrispondere al DDL atteso dal TO-BE. Verificare nell'audit DDL Sprint 0 (comando `\d cons_s_consenso` su DB AS-IS reale).

### 3. Notifica acquisizione — da sincrona ad asincrona

AS-IS: SIA ASR riceve notifica in real-time durante acquisizione.
TO-BE: BATCH-01 notifica entro 5 minuti. Se SIA ha logica temporale che dipende dalla notifica sincrona, adattamento necessario.

---

## Cosa rimane invariato

- **Contratto WSDL outbound** ([[wiki/sources/2019-06-01-webservice-consenso-regionale-v03\|Specifica WebService ConsensoRegionaleAziendale v03 (AS-IS)]]) — confermato da [[wiki/entities/csi-piemonte\|CSI Piemonte]], invariato nel TO-BE
- **Logica fondamentale** — un consenso per tipo/sottotipo/ASR per cittadino
- **Dati di business core** — tipi consenso, ASR, valori SI/NO, codice fiscale
- **Accesso operatori** — RUPAR/IRIDE (PUA) — meccanismo invariato
- **Namespace SOAP** — `http://consprefbe.csi.it/`

---

## Cosa viene dismesso

| Componente                                            | Motivo                                                                |
| ----------------------------------------------------- | --------------------------------------------------------------------- |
| PostgreSQL 9                                          | RETIRED da [[wiki/entities/csi-piemonte\|CSI Piemonte]] — obbligo migrazione a PG17 |
| Stack legacy applicativo                              | Rifacimento completo su [[wiki/concepts/architettura-iaas\|Architettura IaaS]]    |
| Acquisizione sincrona SIA→Regionale come unico canale | Sostituito da CDU-15/16 REST + BATCH-01 asincrono                     |
| Assenza tracciatura storico (`cons_s_consenso` vuota) | TO-BE attiva storicizzazione immutabile                               |

---

## ADR correlati

| ADR | Decisione |
|---|---|
| [ADR-007](ADR-007-batch-01-5min-skip-locked.md) | BATCH-01 5min (cambio paradigma sync → async) |
| [ADR-009](ADR-009-eliminazione-sistemats.md) | Eliminazione SistemaTS (semplificazione stack integrazione) |
| [ADR-010](ADR-010-cdu-01-split.md) | Split CDU-01 (separazione canali auth) |
| [ADR-011](ADR-011-merge-cdu-04-05-cittadino.md) | Merge CDU-04/05 lato Cittadino |
| [ADR-015](ADR-015-storicizzazione-immutabile.md) | Storicizzazione immutabile (attivazione feature `cons_s_consenso`) |
| [ADR-016](ADR-016-scaduto-async-batch-02.md) | SCADUTO async (cambio semantica AS-IS → TO-BE) |
| [ADR-017](ADR-017-lis-terzo-canale.md) | LIS terzo canale |
| [ADR-006](ADR-006-batch-03-pull-cdu-17.md) | BATCH-03 push → CDU-17 PULL (**accepted** — confermato 20/07/2026) |
