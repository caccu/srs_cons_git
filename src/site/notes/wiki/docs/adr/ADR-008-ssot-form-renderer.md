---
{"dg-publish":true,"permalink":"/wiki/docs/adr/adr-008-ssot-form-renderer/","title":"Single Source of Truth — Form Renderer dinamico unico per Cittadino e Operatore","tags":["ui","form-renderer","ssot","cdu-02","cdu-09","mf57","vincolo-architetturale","angular"],"dg-note-properties":{"adr":8,"title":"Single Source of Truth — Form Renderer dinamico unico per Cittadino e Operatore","status":"accepted","date":"2026-05-14","deciders":["Marco Forneris","Exprivia"],"supersedes":[],"superseded-by":[],"tags":["ui","form-renderer","ssot","cdu-02","cdu-09","mf57","vincolo-architetturale","angular"],"related_wiki":["[[wiki/concepts/composizione-dinamica-form-consenso\|Composizione Dinamica Form Consenso — Single Source of Truth]]","[[Gestione Consensi - Applicativo]]","[[ciclo-vita-consenso|Ciclo di Vita del Consenso]]"],"sources":["[[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|Risposte MF SRS v3]] MF57R56"]}}
---


# ADR-008: Single Source of Truth — Form Renderer unico

## Status

`accepted` — vincolo architetturale dichiarato in MF57R56 sulla revisione SRS v3.

> 🔄 **Nota di scope (call CSI 06/08/2026, [[wiki/docs/adr/ADR-021-perimetro-solo-operatore\|ADR-021]]):** il pattern Form Renderer resta valido, ma in questo progetto si applica **solo alla Webapp Operatore** — la Webapp Cittadino non è un deliverable di questo progetto. La motivazione originaria "riuso Citt+Op" non è più applicabile in pratica: non è compito di questo progetto verificare o garantire che la Webapp Cittadino esistente usi lo stesso renderer.

## Context

Il SRS prevede due webapp:
- **app-cittadino** (CDU-02, CDU-03, CDU-04 lato cittadino)
- **app-operatore** (CDU-09, CDU-10, CDU-11 lato operatore PUA)

Entrambe rendono il form di consenso per la stessa configurazione (`tipo_consenso`, `sotto_tipo_consenso`). Senza vincolo esplicito, due implementazioni divergenti del form sono il default — con conseguenze gravi:

- Bug duplicati (e diversamente diagnosticabili)
- Drift UX: cittadino e operatore vedono validazioni o campi diversi per lo stesso consenso
- Difficoltà di evoluzione: ogni nuovo `sotto_tipo_consenso` deve essere aggiunto due volte
- Test moltiplicati

## Decision

Vincolo SSoT: **nessuna divergenza di struttura fra interfaccia Cittadino e Operatore** per lo stesso `sotto_tipo_consenso`.

Implementazione:
- Un singolo componente **Form Renderer dinamico** Angular, condiviso fra le 2 webapp
- Input: `tipo_consenso`, `sotto_tipo_consenso`, `codice_ente` (per aziendali), `modalita` (`citt` | `operatore`)
- Output: form renderizzato + binding bidirezionale con backend
- Componente da aggiungere a SRS §3.3 Componenti software

Differenze ammesse fra Cittadino e Operatore — **solo metadati di tracciatura sorgente** (non struttura form):

| Campo | Cittadino | Operatore |
|---|---|---|
| `fonte_id` | da contesto webapp (es. `WEB_CITT`) | `'PASS'` |
| `login_operazione` | CF cittadino autenticato | login operatore PUA |
| `ruoloop_id` | NULL | ID ruolo operatore PUA |

Configurazione completamente DB-driven (`cons_d_sotto_tipo_cons` + tabelle correlate). Nulla hardcoded a livello UI.

## Consequences

### Positive
- Coerenza UX cittadino/operatore garantita per costruzione
- Singolo punto di evoluzione per ogni nuovo `sotto_tipo_consenso`
- Riduzione superficie test (un solo renderer da testare per regola)
- Architettura aperta a nuovi canali (es. mobile) — ⚠️ LIS **non** rientra in questo caso: acquisisce via integrazione BE esistente, non tramite il Form Renderer (vedi [[wiki/docs/adr/ADR-020-lis-integrazione-be-esistente\|ADR-020]], supersede [[wiki/docs/adr/ADR-017-lis-terzo-canale\|ADR-017]])

### Negative
- Vincola le 2 webapp ad usare la stessa stack frontend (Angular + componente shared)
- Versioning del componente shared deve essere sincronizzato fra le webapp
- Endpoint `GET /api/v1/config/sotto-tipo-consenso/{sotto_tipo}` da implementare

### Neutral
- Cache lato browser opportuna per ridurre fetch ripetuti

## Alternatives considered

| Alternativa                               | Motivo scarto                                                                                  |
| ----------------------------------------- | ---------------------------------------------------------------------------------------------- |
| Due renderer separati cittadino/operatore | Drift garantito; bug doppi; rigetta MF57R56                                                    |
| Server-side rendering                     | Backend Spring Boot deve restare API-only per coerenza con stack [[wiki/docs/adr/ADR-001-stack-tecnologico\|ADR-001-stack-tecnologico]] |
| Form schema standard (JSON Schema / RJSF) | Aggiunge dipendenza esterna; configurazione DB già presente                                    |

## References

- [[wiki/concepts/composizione-dinamica-form-consenso\|Composizione Dinamica Form Consenso — Single Source of Truth]] (concept completo con regole §2-§5)
- [[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|Risposte MF SRS v3]] MF57R56 (+ MF26R25, MF28R27, MF30R29, MF37R36)
- Correlato: [[wiki/docs/adr/ADR-011-merge-cdu-04-05-cittadino\|ADR-011-merge-cdu-04-05-cittadino]] merge CDU-04/CDU-05 lato cittadino (uso pulsante unico abilitato dal renderer dinamico)
