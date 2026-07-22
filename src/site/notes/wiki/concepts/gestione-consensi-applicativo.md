---
{"dg-publish":true,"permalink":"/wiki/concepts/gestione-consensi-applicativo/","title":"Gestione Consensi - Applicativo","tags":["applicativo","gestione-consensi","sanita-piemonte","to-be"],"dg-note-properties":{"title":"Gestione Consensi - Applicativo","aliases":["Gestione Consensi - Applicativo"],"type":"concept","tags":["applicativo","gestione-consensi","sanita-piemonte","to-be"],"created":"2026-05-05","updated":"2026-05-14","sources":["2026-03-02-conspref-srs-v1-revised","2023-09-01-conspref-srs-01-v03","2019-02-01-sfu-gestione-consensi-v1-7"],"related":["[[CSI Piemonte]]","[[exprivia|Exprivia S.p.A.]]","[[wiki/concepts/ciclo-vita-consenso\|Ciclo di Vita del Consenso]]","[[Architettura IaaS]]","[[2026-03-12-pile-tecnologiche-csi|Pile Tecnologiche CSI Piemonte]]","[[wiki/concepts/composizione-dinamica-form-consenso\|Composizione Dinamica Form Consenso — Single Source of Truth]]","[[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|analysis-2026-05-14-risposte-mf-srs-v3]]"]}}
---


# Gestione Consensi — Applicativo

Sistema centrale per la raccolta, gestione e consultazione dei consensi sanitari dei cittadini piemontesi. Parte dell'ecosistema Sanità Elettronica Regione Piemonte.

---

## Tre livelli di consenso

| Livello | Ambito | Esempi |
|---|---|---|
| Nazionale | Tutto il territorio | FSE/INI (apertura fascicolo) |
| Regionale | Regione Piemonte | Stratificazione, cronicità, telemedicina, reti patologia |
| Aziendale | Singola ASR | Dossier Clinico, ROL referti, trattamento dati personali |

**Regola chiave:** Consenso regionale = 1 scelta → propagata a tutte le ASR. Consenso aziendale = 1 scelta per ogni ASR separatamente.

---

## Canali di acquisizione consenso (MF3R1, MF4R1)

Il cittadino può esprimere il proprio consenso attraverso **tre canali** distinti:

| Canale | Descrizione | Profilo accesso |
|---|---|---|
| Webapp Cittadino | Webapp dedicata SPID/CIE | Cittadino autenticato |
| Webapp Operatore | Operatore opera per conto dell'assistito | Operatore PUA (RUPAR/IRIDE) |
| **LIS** | Acquisizione presso il Laboratorio (canale di front-office) | Operatore LIS dedicato |

> Decisione MF4R1: aggiungere didascalia nel diagramma di contesto: "Consensi esprimibili anche presso LIS oltre webapp cittadino e Operatore". Aggiornare SRS §1/§2.

---

## Profili utente (aggiornato MF7, MF16)

| Profilo                            | Accesso                                                          | CDU                    | Profilo applicativo Configuratore?      |
| ---------------------------------- | ---------------------------------------------------------------- | ---------------------- | --------------------------------------- |
| **Cittadino**                      | SPID/CIE via [[wiki/concepts/gasp-salute\|GASP Salute]] su **webapp dedicata** | CDU-01b, CDU-02÷CDU-06 | ❌ **NO** — non gestito da Configuratore |
| Operatore Sanitario/Amministrativo | PUA / RUPAR/IRIDE                                                | CDU-01a, CDU-07÷CDU-11 | Sì                                      |
| Operatore di Back Office           | PUA / RUPAR/IRIDE                                                | CDU-12÷CDU-14          | Sì                                      |
| SIA Aziendale                      | API REST (OAuth2 Bearer JWT)                                     | CDU-15, CDU-16, CDU-17 | N/A — autenticazione machine-to-machine |

### Nota su profilo Cittadino (MF7R5)

> Cittadino **NON è profilo applicativo del Configuratore**. Accede a webapp dedicata (SPID/CIE). Il "profilo" logico interno serve solo a distinguere se l'utente sta agendo per **sé stesso** o per un **delegante** (selezionato tramite pulsante "Deleghe").

Implicazione: nelle tabelle dei profili in SRS aggiungere colonna "NOTE" che marca esplicitamente questa differenza.

---

## CDU-01 split (MF16R15, MF18R17)

Il caso d'uso di autenticazione viene diviso in due sotto-scenari espliciti:

### CDU-01a — Accesso Operatore

| Aspetto | Dettaglio |
|---|---|
| Accesso | RUPAR / IRIDE / SPID via PUA |
| Selezione profilo | Sì — il sistema mostra la selezione del profilo operatore |
| Precondizioni | Operatore censito in Configuratore Regionale |

### CDU-01b — Accesso Cittadino

| Aspetto | Dettaglio |
|---|---|
| Accesso | SPID / CIE su **webapp separata** |
| Profilazione iniziale | Nessuna |
| Precondizioni | Cittadino con identità digitale attiva |

---

## Flusso Deleghe (MF20R19, MF22R21)

> La webapp del cittadino mostra **sempre** il cruscotto dell'utente autenticato. Se il cittadino vuole operare per conto di un delegante, clicca il pulsante **"Deleghe"**, che mostra l'elenco dei deleganti attivi. Selezionando un delegante, il sistema carica il cruscotto consensi di quel soggetto.

```
Pulsante «Deleghe» → elenco deleganti (servizio Gestione Deleghe) → selezione → cruscotto consensi del delegante
```

**Nota implementativa (MF22R21):** questo scenario è già attivo in produzione. Implementazione segue pattern esistente.

Variante 6.1.3 [PROPOSTA]: se il servizio Gestione Deleghe non risponde o restituisce un errore, il sistema impedisce la selezione del profilo delegato e mostra un messaggio di avviso. L'utente può comunque operare solo per sé stesso.

---

## Funzionalità principali TO-BE

### Area Cittadino (webapp dedicata)

- **CDU-02** Consultazione cruscotto consensi: mostra **tutti i tipi configurati** (regionale=1 record, aziendale=N record per azienda) anche quelli non ancora espressi (MF26R25). Vedi [[wiki/concepts/composizione-dinamica-form-consenso\|Composizione Dinamica Form Consenso — Single Source of Truth]].
- **CDU-03** Rilascio nuovo consenso (con visualizzazione informativa PDF)
- **CDU-04** Modifica consenso — **ingloba anche il cambio valore** (CDU-05 non separato lato cittadino, MF45R44). Pulsante unico "Salva" (MF37R36).
- **CDU-06** Download PDF — scope ridotto (MF47R46):
  - Cittadino stampa PDF della **sola informativa accettata**
  - **Senza** firma digitale
  - **Senza** valore consenso espresso
  - Struttura ulteriore: `[PROPOSTA]` da concordare (MF49R48, MF51R50)

### Area Operatore

- Ricerca assistito via **AURA** (FindProfiliAnagrafici + getProfiloSanitario). Se CF non trovato → messaggio "La ricerca con il filtro fornito non ha prodotto risultati". **Nessuna chiamata a SistemaTS** (MF53R52, MF55R54).
- Gestione consensi per conto dell'assistito — CDU-09 (rilascio), CDU-10 (modifica), CDU-11 (cambio valore). Composizione pagina **identica al lato cittadino**, motore Form Renderer unico (MF57R56). Vedi [[wiki/concepts/composizione-dinamica-form-consenso\|Composizione Dinamica Form Consenso — Single Source of Truth]].
- Tracciatura: `fonte_id='PASS'`, `login_operazione`, `ruoloop_id` valorizzati con dati operatore.

### Area Back Office

- Configurazione tipi consenso (parametri dinamici — input del Form Renderer)
- Gestione informative (upload PDF, versioning)
- Gestione enti ed endpoint (**CDU-14**) — trigger dell'allineamento. Vedi [[wiki/concepts/alternativa-batch-03-pull\|Alternativa BATCH-03 — PULL CDU-17 (centro stella)]] per il modello PULL sostituto del BATCH-03 push.

### API per SIA

- `GET /api/v1/consensi/stato` (CDU-15)
- `GET /api/v1/configurazione/{codiceEnte}` (CDU-16)
- `GET /api/v1/consensi/snapshot` (CDU-17, ✅ confermato 20/07/2026 — vedi [[wiki/concepts/alternativa-batch-03-pull\|Alternativa BATCH-03 — PULL CDU-17 (centro stella)]])

Sicurezza per tutti gli endpoint SIA: vedi [[wiki/concepts/sicurezza-cdu-15-16\|Sicurezza CDU-15-16 — Modello Autorizzazione per Ente]].

---

## Notifica al cittadino post-acquisizione (MF33R31)

> Notifica cittadino/delegato via **Notificatore di Deleghe** (NON UNP) parte **SOLO dopo conferma notifica alle aziende** (stato = COMPLETATO).

Dipendenza temporale: la notifica al cittadino arriva solo a valle del completamento di BATCH-01. Vedi [[wiki/concepts/batch-processes\|Processi Batch — BATCH-01, BATCH-02, BATCH-03]] e [[wiki/concepts/sistemi-esterni-integrati\|Sistemi Esterni Integrati]].

---

## Ciclo di vita consenso

Vedi [[wiki/concepts/ciclo-vita-consenso\|Ciclo di Vita del Consenso]]. Aggiornato con transizioni dirette ATTIVO↔NEGATO (MF11/MF14) e terminologia UI "acconsento/nego" (MF9).

## Architettura

Vedi [[wiki/concepts/architettura-iaas\|Architettura IaaS]]. Nuovo componente da aggiungere in SRS §3.3: **Form Renderer dinamico** (motore di rendering form-consenso unico, condiviso fra webapp Cittadino e webapp Operatore — vincolo SSoT MF57R56).

---

## Storia documento

| Versione | Anno | Documento |
|---|---|---|
| AS-IS | 2019 | [[wiki/sources/2019-02-01-sfu-gestione-consensi-v1-7\|Studio Funzionale Gestione Consensi SoL V1.7 (AS-IS 2019)]] |
| Requisiti committente | 2023 | [[wiki/sources/2023-09-01-conspref-srs-01-v03\|CONSPREF-SRS-01-V03 Requisiti Gestione Consensi]] |
| SRS TO-BE | 2026 | [[wiki/sources/2026-03-02-conspref-srs-v1-revised\|CONSPREF-SRS-V1.0 revised bozza v2]] |
| Revisione cliente v3 | 2026-05 | `raw/CONSPREF-SRS-V1.0-revised_bozza_v3_CSI_lavorazione.pdf` — 69 commenti, ~30 risposte MF. Sintesi: [[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|analysis-2026-05-14-risposte-mf-srs-v3]] |

---

## ADR correlati

| ADR | Decisione |
|---|---|
| [ADR-001](ADR-001-stack-tecnologico.md) | Stack tecnologico |
| [ADR-010](ADR-010-cdu-01-split.md) | Split CDU-01 in CDU-01a Operatore + CDU-01b Cittadino |
| [ADR-011](ADR-011-merge-cdu-04-05-cittadino.md) | Merge CDU-04/05 lato Cittadino (pulsante unico) |
| [ADR-008](ADR-008-ssot-form-renderer.md) | SSoT Form Renderer dinamico |
| [ADR-017](ADR-017-lis-terzo-canale.md) | LIS terzo canale di acquisizione |
| [ADR-019](ADR-019-cdu-06-pdf-scope-ridotto.md) | CDU-06 PDF scope ridotto |
