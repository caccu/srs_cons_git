---
{"dg-publish":true,"permalink":"/wiki/concepts/gestione-consensi-applicativo/","title":"Gestione Consensi - Applicativo","tags":["applicativo","gestione-consensi","sanita-piemonte","to-be"],"dg-note-properties":{"title":"Gestione Consensi - Applicativo","aliases":["Gestione Consensi - Applicativo"],"type":"concept","tags":["applicativo","gestione-consensi","sanita-piemonte","to-be"],"created":"2026-05-05","updated":"2026-05-14","sources":["2026-03-02-conspref-srs-v1-revised","2023-09-01-conspref-srs-01-v03","2019-02-01-sfu-gestione-consensi-v1-7"],"related":["[[CSI Piemonte]]","[[exprivia|Exprivia S.p.A.]]","[[wiki/concepts/ciclo-vita-consenso\|Ciclo di Vita del Consenso]]","[[Architettura IaaS]]","[[2026-03-12-pile-tecnologiche-csi|Pile Tecnologiche CSI Piemonte]]","[[wiki/concepts/composizione-dinamica-form-consenso\|Composizione Dinamica Form Consenso — Single Source of Truth]]","[[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|analysis-2026-05-14-risposte-mf-srs-v3]]"]}}
---


# Gestione Consensi — Applicativo

Sistema centrale per la raccolta, gestione e consultazione dei consensi sanitari dei cittadini piemontesi. Parte dell'ecosistema Sanità Elettronica Regione Piemonte.

> 🔴 **Perimetro progetto (call CSI 06/08/2026, [[wiki/docs/adr/ADR-021-perimetro-solo-operatore\|ADR-021]]):** questo progetto sviluppa **solo la Webapp Operatore**. La Webapp Cittadino esiste ma è **fuori dal perimetro di sviluppo** — non è un deliverable di questo progetto. Le sezioni marcate ❌ **OUT** sotto descrivono funzionalità cittadino non costruite da questo progetto (mantenute come contesto/storia, non come backlog).

---

## Tre livelli di consenso

| Livello | Ambito | Esempi |
|---|---|---|
| Nazionale | Tutto il territorio | FSE/INI (apertura fascicolo) |
| Regionale | Regione Piemonte | Stratificazione, cronicità, telemedicina, reti patologia |
| Aziendale | Singola ASR | Dossier Clinico, ROL referti, trattamento dati personali |

**Regola chiave:** Consenso regionale = 1 scelta → propagata a tutte le ASR. Consenso aziendale = 1 scelta per ogni ASR separatamente.

---

## Canali di acquisizione consenso (MF3R1, MF4R1 — rivisto ADR-020, ADR-021)

Il consenso è esprimibile attraverso **due canali** UI a livello di sistema:

| Canale | Descrizione | Profilo accesso | Scope di sviluppo |
|---|---|---|---|
| Webapp Cittadino | Webapp dedicata SPID/CIE | Cittadino autenticato | ❌ **OUT** — esiste ma fuori dal perimetro di questo progetto ([[wiki/docs/adr/ADR-021-perimetro-solo-operatore\|ADR-021]]) |
| Webapp Operatore | Operatore opera per conto dell'assistito | Operatore PUA (RUPAR/IRIDE) | ✅ **IN** — unico deliverable di questo progetto |

> ⚠️ **Rivisto (call CSI 06/08/2026, chiude INT-03):** LIS **non è un terzo canale di acquisizione UI**. La decisione MF4R1 ("Consensi esprimibili anche presso LIS") si riferisce a un'**integrazione BE già presente nel codice sorgente AS-IS** ([[wiki/concepts/sistemi-esterni-integrati\|Sistemi Esterni Integrati]] §LIS), non a un nuovo canale da progettare. Attività TO-BE: verificare e migrare l'integrazione esistente al nuovo stack. Vedi [[wiki/docs/adr/ADR-020-lis-integrazione-be-esistente\|ADR-020]] (supersede [[wiki/docs/adr/ADR-017-lis-terzo-canale\|ADR-017]]).
>
> 🔴 **Perimetro (call CSI 06/08/2026):** di questi due canali, solo la **Webapp Operatore** è in scope di sviluppo di questo progetto. Vedi [[wiki/docs/adr/ADR-021-perimetro-solo-operatore\|ADR-021]].

---

## Profili utente (aggiornato MF7, MF16 — perimetro ADR-021, profilo unico 06/08/2026)

> ✅ **Chiarito (call CSI 06/08/2026):** **un solo profilo Operatore**, unico, copre **tutte** le nuove funzionalità da sviluppare (CDU-01a, CDU-05, CDU-07÷CDU-14). **Nessun nuovo profilo da creare** in PUA/Configuratore — corregge la precedente distinzione "Operatore Sanitario/Amministrativo" vs "Operatore di Back Office" come profili applicativi separati.

| Profilo             | Accesso                                                          | CDU                              | Profilo applicativo Configuratore?      | Scope sviluppo |
| -------------------- | ---------------------------------------------------------------- | --------------------------------- | --------------------------------------- | --- |
| **Cittadino**         | SPID/CIE via [[wiki/concepts/gasp-salute\|GASP Salute]] su **webapp dedicata** | CDU-01b, CDU-02÷CDU-06            | ❌ **NO** — non gestito da Configuratore | ❌ **OUT** — [[wiki/docs/adr/ADR-021-perimetro-solo-operatore\|ADR-021]] |
| **Operatore** (unico) | PUA / RUPAR/IRIDE                                                | CDU-01a, CDU-05, CDU-07÷CDU-14    | Sì — **1 solo profilo**                 | ✅ IN — unico deliverable |
| SIA Aziendale         | API REST (OAuth2 Bearer JWT)                                     | CDU-15, CDU-16, CDU-17            | N/A — autenticazione machine-to-machine | ✅ IN |

### Nota su profilo Cittadino (MF7R5)

> Cittadino **NON è profilo applicativo del Configuratore**. Accede a webapp dedicata (SPID/CIE). Il "profilo" logico interno serve solo a distinguere se l'utente sta agendo per **sé stesso** o per un **delegante** (selezionato tramite pulsante "Deleghe").

Implicazione: nelle tabelle dei profili in SRS aggiungere colonna "NOTE" che marca esplicitamente questa differenza.

---

## CDU-01 split (MF16R15, MF18R17 — perimetro ADR-021)

Il caso d'uso di autenticazione viene diviso in due sotto-scenari espliciti:

### CDU-01a — Accesso Operatore ✅ IN scope

| Aspetto | Dettaglio |
|---|---|
| Accesso | RUPAR / IRIDE / SPID via PUA |
| Selezione profilo | ❌ No — **un solo profilo Operatore** (call CSI 06/08/2026), niente selezione multipla |
| Precondizioni | Operatore censito in Configuratore Regionale |

### CDU-01b — Accesso Cittadino ❌ OUT scope

> 🔴 **Fuori dal perimetro di sviluppo** (call CSI 06/08/2026, [[wiki/docs/adr/ADR-021-perimetro-solo-operatore\|ADR-021]]) — la Webapp Cittadino esiste ma non è un deliverable di questo progetto.

| Aspetto | Dettaglio |
|---|---|
| Accesso | SPID / CIE su **webapp separata** |
| Profilazione iniziale | Nessuna |
| Precondizioni | Cittadino con identità digitale attiva |

---

## Flusso Deleghe (MF20R19, MF22R21) ❌ OUT scope

> 🔴 **Fuori dal perimetro di sviluppo** (call CSI 06/08/2026, [[wiki/docs/adr/ADR-021-perimetro-solo-operatore\|ADR-021]]) — scenario interamente sulla Webapp Cittadino, non un deliverable di questo progetto. Mantenuto come contesto storico.

> La webapp del cittadino mostra **sempre** il cruscotto dell'utente autenticato. Se il cittadino vuole operare per conto di un delegante, clicca il pulsante **"Deleghe"**, che mostra l'elenco dei deleganti attivi. Selezionando un delegante, il sistema carica il cruscotto consensi di quel soggetto.

```
Pulsante «Deleghe» → elenco deleganti (servizio Gestione Deleghe) → selezione → cruscotto consensi del delegante
```

**Nota implementativa (MF22R21):** questo scenario è già attivo in produzione. Implementazione segue pattern esistente.

Variante 6.1.3 [PROPOSTA]: se il servizio Gestione Deleghe non risponde o restituisce un errore, il sistema impedisce la selezione del profilo delegato e mostra un messaggio di avviso. L'utente può comunque operare solo per sé stesso.

---

## Funzionalità principali TO-BE

### Area Cittadino (webapp dedicata) ❌ OUT scope

> 🔴 **Fuori dal perimetro di sviluppo** (call CSI 06/08/2026, [[wiki/docs/adr/ADR-021-perimetro-solo-operatore\|ADR-021]]). La Webapp Cittadino esiste ma questi CDU non sono un deliverable di questo progetto. Sezione mantenuta come contesto/storia.

- **CDU-02** Consultazione cruscotto consensi: mostra **tutti i tipi configurati** (regionale=1 record, aziendale=N record per azienda) anche quelli non ancora espressi (MF26R25). Vedi [[wiki/concepts/composizione-dinamica-form-consenso\|Composizione Dinamica Form Consenso — Single Source of Truth]].
- **CDU-03** Rilascio nuovo consenso (con visualizzazione informativa PDF)
- **CDU-04** Modifica consenso — **ingloba anche il cambio valore** (CDU-05 non separato lato cittadino, MF45R44). Pulsante unico "Salva" (MF37R36). Decisione [[wiki/docs/adr/ADR-011-merge-cdu-04-05-cittadino\|ADR-011]], superseded.
- **CDU-06** Download PDF — scope ridotto (MF47R46). Decisione [[wiki/docs/adr/ADR-019-cdu-06-pdf-scope-ridotto\|ADR-019]], superseded:
  - Cittadino stampa PDF della **sola informativa accettata**
  - **Senza** firma digitale
  - **Senza** valore consenso espresso
  - Struttura ulteriore: `[PROPOSTA]` da concordare (MF49R48, MF51R50)

### Area Operatore ✅ IN scope

- Ricerca assistito via **AURA** (FindProfiliAnagrafici + getProfiloSanitario). Se CF non trovato → messaggio "La ricerca con il filtro fornito non ha prodotto risultati". **Nessuna chiamata a SistemaTS** (MF53R52, MF55R54).
- Gestione consensi per conto dell'assistito — CDU-09 (rilascio), CDU-10 (modifica), CDU-11 (cambio valore). Motore Form Renderer ([[wiki/concepts/composizione-dinamica-form-consenso\|Composizione Dinamica Form Consenso — Single Source of Truth]]) — in questo progetto usato **solo dalla Webapp Operatore** (vedi [[wiki/docs/adr/ADR-021-perimetro-solo-operatore\|ADR-021]]; il riuso con la Webapp Cittadino non è più in scope).
- Tracciatura: `fonte_id='PASS'`, `login_operazione`, `ruoloop_id` valorizzati con dati operatore.

### Area Back Office ✅ IN scope

> Nota (06/08/2026): funzioni descritte come area distinta, ma sotto lo **stesso profilo Operatore unico** — non un secondo profilo Configuratore separato.

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

Vedi [[wiki/concepts/architettura-iaas\|Architettura IaaS]]. Componente da aggiungere in SRS §3.3: **Form Renderer dinamico** (motore di rendering form-consenso — vincolo SSoT MF57R56, nato per uso condiviso Cittadino+Operatore, in questo progetto usato **solo dalla Webapp Operatore**, vedi [[wiki/docs/adr/ADR-021-perimetro-solo-operatore\|ADR-021]]).

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
| [ADR-010](ADR-010-cdu-01-split.md) | Split CDU-01 in CDU-01a Operatore + CDU-01b Cittadino (nota scope: solo 01a in scope) |
| [ADR-011](ADR-011-merge-cdu-04-05-cittadino.md) | Merge CDU-04/05 lato Cittadino (pulsante unico) — **superseded** da ADR-021 |
| [ADR-008](ADR-008-ssot-form-renderer.md) | SSoT Form Renderer dinamico (nota scope: solo Webapp Operatore) |
| [ADR-020](ADR-020-lis-integrazione-be-esistente.md) | LIS/RIS integrazione BE esistente (supersede [ADR-017](ADR-017-lis-terzo-canale.md)) |
| [ADR-019](ADR-019-cdu-06-pdf-scope-ridotto.md) | CDU-06 PDF scope ridotto — **superseded** da ADR-021 |
| [ADR-021](ADR-021-perimetro-solo-operatore.md) | Perimetro progetto: solo Webapp Operatore |
