---
{"dg-publish":true,"permalink":"/wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3/","title":"Risposte MF — Revisione SRS v3 lavorazione (69 commenti)","tags":["revisione","srs-v3","risposte-mf","marco-forneris","tracker","decisioni","propagazione-wiki"],"dg-note-properties":{"title":"Risposte MF — Revisione SRS v3 lavorazione (69 commenti)","aliases":["Risposte MF — Revisione SRS v3 lavorazione (69 commenti)"],"type":"analysis","tags":["revisione","srs-v3","risposte-mf","marco-forneris","tracker","decisioni","propagazione-wiki"],"created":"2026-05-14","updated":"2026-05-14","sources":["2026-03-02-conspref-srs-v1-revised"],"related":["[[sicurezza-cdu-15-16|Sicurezza CDU-15-16 — Modello Autorizzazione per Ente]]","[[alternativa-batch-03-pull|Alternativa BATCH-03 — PULL CDU-17 (centro stella)]]","[[composizione-dinamica-form-consenso|Composizione Dinamica Form Consenso — Single Source of Truth]]","[[ciclo-vita-consenso|Ciclo di Vita del Consenso]]","[[Gestione Consensi - Applicativo]]","[[Sistemi Esterni Integrati]]","[[batch-processes|Processi Batch — BATCH-01, BATCH-02, BATCH-03]]","[[analysis-gap-as-is-to-be|Analisi Gap AS-IS → TO-BE — Gestione Consensi]]"]}}
---


# Risposte MF — Revisione SRS v3 lavorazione

**File sorgente:** `raw/CONSPREF-SRS-V1.0-revised_bozza_v3_CSI_lavorazione.pdf`
**Commenti totali:** 69 (SC = Sicurezza/Customer, MB = Marco Bianchi reviewer, TR = Tecnico/Reviewer)
**Risposte MF (Marco Forneris/Exprivia):** ~30
**Data analisi:** 2026-05-14

Le risposte MF nel PDF sono **decisioni autoritative** del responsabile tecnico Exprivia. Questa analisi consolida tutte le decisioni e mappa la propagazione sulla wiki.

> **Nota numerazione:** la v3_lavorazione rinumera i commenti rispetto alla v3. Mapping principali: TR30 → TR58 (sicurezza CDU-15/16), TR34 → TR68 (alternativa BATCH-03). Wiki riporta entrambi gli ID.

> ⚠️ **Superato in parte (call CSI 06/08/2026, [[wiki/docs/adr/ADR-021-perimetro-solo-operatore\|ADR-021]]):** il progetto sviluppa **solo la Webapp Operatore**. Le decisioni sotto su CDU-01b, CDU-02, CDU-03, CDU-04 (lato Cittadino), CDU-06 e GASP Salute restano come registro storico ma non sono più deliverable di questo progetto (temi A/C/D, righe MF16R15, MF26R25, MF39R38, MF45R44 parte cittadino, MF47R46, MF49R48, MF51R50).

---

## Tabella maestra — 30 risposte MF

| ID MF        | Tema                        | Decisione (sintesi)                                                                                                                                                                                                         | Pagina wiki                                                                                                                  | Stato           |
| ------------ | --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- | --------------- |
| MF3R1, MF4R1 | A — Canali acquisizione     | Cittadino può esprimere consenso anche presso **LIS** (oltre webapp citt + operatore). Aggiungere didascalia diagramma ⚠️ **Rivisto 06/08/2026:** LIS non è un terzo canale UI ma un'integrazione BE esistente — vedi [[wiki/docs/adr/ADR-020-lis-integrazione-be-esistente\|ADR-020]]                                                                                                    | [[wiki/concepts/gestione-consensi-applicativo\|Gestione Consensi - Applicativo]], [[wiki/concepts/sistemi-esterni-integrati\|Sistemi Esterni Integrati]] | ⚠️ superato               |
| MF7R5        | A — Profili                 | Cittadino NON è profilo applicativo del Configuratore. Accede webapp dedicata SPID/CIE. Profilo logico interno solo per «sé stesso» vs «delegante»                                                                          | [[wiki/concepts/gestione-consensi-applicativo\|Gestione Consensi - Applicativo]]                                                           | ✅               |
| MF9R8        | A — UI termini              | Sostituire "Accettato/Negato" con **"acconsento/nego"** in tutta la nota tecnica 5.2. Mantenere mapping tecnico ATTIVO/NEGATO invariato                                                                                     | [[wiki/concepts/ciclo-vita-consenso\|ciclo-vita-consenso]]                                                                                                      | ✅               |
| MF11R10      | B — Stati                   | Aggiungere transizione **ATTIVO → NEGATO** (via webapp citt o operatore) senza richiedere nuova informativa                                                                                                                 | [[wiki/concepts/ciclo-vita-consenso\|ciclo-vita-consenso]]                                                                                                      | ✅               |
| MF14R12      | B — Stati                   | Aggiungere transizione **NEGATO → ATTIVO** (via webapp citt o operatore) senza richiedere nuova informativa                                                                                                                 | [[wiki/concepts/ciclo-vita-consenso\|ciclo-vita-consenso]]                                                                                                      | ✅               |
| MF16R15      | A — CDU-01                  | Dividere CDU-01: **CDU-01a Operatore** (RUPAR/IRIDE/SPID + profilazione) e **CDU-01b Cittadino** (SPID/CIE webapp separata, no profilazione)                                                                                | [[wiki/concepts/gestione-consensi-applicativo\|Gestione Consensi - Applicativo]], [[wiki/concepts/gasp-salute\|GASP Salute]]                             | ✅               |
| MF18R17      | A — CDU-01                  | Dividere precondizioni CDU-01 per scenario (a/b)                                                                                                                                                                            | [[wiki/concepts/gestione-consensi-applicativo\|Gestione Consensi - Applicativo]]                                                           | ✅               |
| MF20R19      | A — Deleghe                 | Webapp cittadino mostra sempre utente autenticato. Pulsante "Deleghe" → elenco deleganti → selezione carica cruscotto del delegante                                                                                         | [[wiki/concepts/gestione-consensi-applicativo\|Gestione Consensi - Applicativo]], [[wiki/concepts/sistemi-esterni-integrati\|Sistemi Esterni Integrati]] | ✅               |
| MF22R21      | A — Deleghe                 | Nota implementativa: scenario deleghe già attivo in produzione                                                                                                                                                              | [[wiki/concepts/gestione-consensi-applicativo\|Gestione Consensi - Applicativo]]                                                           | ✅               |
| MF24R23      | H — Riferimenti CDU-14      | Aggiungere riferimento esplicito a 6.14 CDU-14 e ALG01 per la logica di selezione endpoint nel flusso flag_online check                                                                                                     | [[wiki/concepts/batch-processes\|batch-processes]]                                                                                                          | ✅               |
| MF26R25      | C — CDU-02                  | Sistema mostra TUTTI i tipi configurati. Regionale=1 record con stato globale. Aziendale=1 record per azienda con stato relativo                                                                                            | [[wiki/concepts/composizione-dinamica-form-consenso\|composizione-dinamica-form-consenso]]                                                                                      | ✅               |
| MF28R27      | C — Informativa             | Consensi aziendali: visualizzare informativa singola azienda **prima** di raccogliere consenso per quell'azienda                                                                                                            | [[wiki/concepts/composizione-dinamica-form-consenso\|composizione-dinamica-form-consenso]]                                                                                      | ✅               |
| MF30R29      | C — Dinamica pagina         | Pagina espressione consenso composta dinamicamente: valori ammessi, domande opzionali, flag richiesto — letti da config DB                                                                                                  | [[wiki/concepts/composizione-dinamica-form-consenso\|composizione-dinamica-form-consenso]]                                                                                      | ✅               |
| MF33R31      | G — Notifica                | Notifica cittadino/delegato via **Notificatore di Deleghe** (NON UNP). Parte SOLO dopo conferma notifica aziende (stato=COMPLETATO)                                                                                         | [[wiki/concepts/batch-processes\|batch-processes]], [[wiki/concepts/sistemi-esterni-integrati\|Sistemi Esterni Integrati]]                                                | ✅               |
| MF35R34      | C — Valori dinamici         | Valori consenso letti dinamicamente da DB per tipo. Composizione pagina dinamica come §6.2                                                                                                                                  | [[wiki/concepts/composizione-dinamica-form-consenso\|composizione-dinamica-form-consenso]]                                                                                      | ✅               |
| MF37R36      | C — UI cittadino            | Webapp cittadino: pulsante unico **«Salva»**. Operazioni distinte (rilascio vs modifica) gestite internamente, non esposte UI                                                                                               | [[wiki/concepts/composizione-dinamica-form-consenso\|composizione-dinamica-form-consenso]]                                                                                      | ✅               |
| MF39R38      | C — Modifica                | Pagina modifica consenso CDU-04 segue stesse regole composizione dinamica §6.2                                                                                                                                              | [[wiki/concepts/composizione-dinamica-form-consenso\|composizione-dinamica-form-consenso]]                                                                                      | ✅               |
| MF41R40      | C — Modifica                | Idem (rinforzo MF39)                                                                                                                                                                                                        | [[wiki/concepts/composizione-dinamica-form-consenso\|composizione-dinamica-form-consenso]]                                                                                      | ✅               |
| MF43R42      | C — Modifica valori         | Valori consenso letti dinamicamente da DB per tipo (rinforzo)                                                                                                                                                               | [[wiki/concepts/composizione-dinamica-form-consenso\|composizione-dinamica-form-consenso]]                                                                                      | ✅               |
| MF45R44      | C — CDU-04/05 merge         | Webapp Cittadino: CDU-05 NON caso d'uso separato → inglobato in CDU-04. Distinzione CDU-04/05 rilevante solo per Operatore e logica interna                                                                                 | [[wiki/concepts/gestione-consensi-applicativo\|Gestione Consensi - Applicativo]], [[wiki/concepts/composizione-dinamica-form-consenso\|composizione-dinamica-form-consenso]]                 | ✅               |
| MF47R46      | D — CDU-06 PDF              | Scope ridotto: Cittadino stampa PDF **solo informativa accettata**, senza firma digitale, senza valore consenso espresso                                                                                                    | [[wiki/concepts/gestione-consensi-applicativo\|Gestione Consensi - Applicativo]]                                                           | ✅               |
| MF49R48      | D — CDU-06 PDF              | Struttura PDF era proposta implementativa: marcata [PROPOSTA] da concordare                                                                                                                                                 | [[wiki/concepts/gestione-consensi-applicativo\|Gestione Consensi - Applicativo]]                                                           | ✅               |
| MF51R50      | D — CDU-06 PDF              | [PROPOSTA] da concordare                                                                                                                                                                                                    | [[wiki/concepts/gestione-consensi-applicativo\|Gestione Consensi - Applicativo]]                                                           | ✅               |
| MF53R52      | E — CDU-07 AURA             | Sistema invoca FindProfiliAnagrafici + getProfiloSanitario di AURA. Se CF non trovato → "La ricerca con il filtro fornito non ha prodotto risultati". **Nessuna chiamata SistemaTS**                                        | [[wiki/concepts/sistemi-esterni-integrati\|Sistemi Esterni Integrati]]                                                                     | ✅               |
| MF55R54      | E — CDU-07 cleanup          | **ELIMINARE tutti i riferimenti fallback SistemaTS** in 6.7                                                                                                                                                                 | [[wiki/concepts/sistemi-esterni-integrati\|Sistemi Esterni Integrati]]                                                                     | ✅               |
| MF57R56      | F — SSoT operatore          | Pagina operatore (CDU-09/10/11) composizione dinamica IDENTICA al cittadino. Single Source of Truth. **Form Renderer unico** condiviso fra le 2 webapp. Differenze solo `fonte_id='PASS'`, `login_operazione`, `ruoloop_id` | [[wiki/concepts/composizione-dinamica-form-consenso\|composizione-dinamica-form-consenso]]                                                                                      | ✅               |
| MF59-62R58   | I — Sicurezza (ex TR30)     | **TR58 = ex TR30.** CDU-15/16: no API Manager, JWT + tabella `cons_t_client_ente` + filter Spring Security. Già in wiki                                                                                                     | [[wiki/concepts/sicurezza-cdu-15-16\|sicurezza-cdu-15-16]]                                                                                                      | ✅               |
| MF64R63      | H — Scheduling              | Aggiungere tabella transizioni stato `cons_t_notifica`/`cons_t_batch`. BATCH-01 **5 minuti** con `SELECT FOR UPDATE SKIP LOCKED` su `cons_t_notifica`. Sostituisce AS-IS 30 minuti                                          | [[wiki/concepts/batch-processes\|batch-processes]]                                                                                                          | ✅               |
| MF66R65      | H — BATCH-02 SQL            | SQL per determinare `nuova_d_informativa_id`: SELECT informativa corrente non scaduta con ORDER BY data_decorrenza DESC LIMIT 1                                                                                             | [[wiki/concepts/batch-processes\|batch-processes]]                                                                                                          | ✅               |
| MF69R68      | I — BATCH-03 PULL (ex TR34) | **TR68 = ex TR34.** Sostituzione BATCH-03 push con PULL CDU-17. Già in wiki                                                                                                                                                 | [[wiki/concepts/alternativa-batch-03-pull\|alternativa-batch-03-pull]]                                                                                                | ✅ **confermato dal committente 20/07/2026** ([ADR-006] **accepted**) |

---

## Commenti senza risposta MF

| ID | Cosa cliente chiede | Stato | Azione |
|---|---|---|---|
| SC67 | "da approfondire e verificare meglio" su INSERT cons_t_consenso da batch (BATCH-02 ALG02 storicizzazione) | 🟠 Aperto | Discutere con CSI/MF prima di chiudere SRS |
| MF3R1 (vuoto) | LIS — la risposta utile è in MF4R1 | Coperto da MF4R1 | — |
| MF13R12 (vuoto) | Stati — la risposta utile è in MF14R12 | Coperto da MF14R12 | — |

---

## Dettaglio per tema

### Tema A — UI terminologia & profili Cittadino

**Decisioni chiave:**

1. **LIS come canale aggiuntivo (MF3, MF4)** — Il cittadino può esprimere consenso anche presso il LIS (Laboratorio Informazioni Sanitarie? — da chiarire acronimo) oltre alla webapp cittadino e alla via operatore. Implicazione: SRS §1/§2 diagramma di contesto deve riflettere 3 canali di acquisizione, non 2.

> ⚠️ **Superato (call CSI 06/08/2026, chiude INT-03):** non esiste un terzo canale di acquisizione. LIS acquisisce tramite integrazione BE già presente nel codice sorgente AS-IS, da verificare e migrare — non un nuovo canale UI. SRS §1/§2 resta a **2 canali**. Vedi [[wiki/docs/adr/ADR-020-lis-integrazione-be-esistente\|ADR-020]] (supersede [[wiki/docs/adr/ADR-017-lis-terzo-canale\|ADR-017]]).

2. **Cittadino NON profilo Configuratore (MF7)** — Webapp dedicata SPID/CIE, distinta da PUA/Configuratore. Il "profilo" cittadino è una **distinzione logica interna** (per discriminare se l'utente sta agendo per sé o per un delegante via pulsante Deleghe), non un profilo applicativo registrato nel Configuratore Regionale.

3. **UI terminologia "acconsento"/"nego" (MF9)** — Termini visibili all'utente. Mapping tecnico immutato: `acconsento` → `ATTIVO`, `nego` → `NEGATO`. La modifica è solo UI; le tabelle DB e API REST continuano a usare `ATTIVO`/`NEGATO`.

4. **CDU-01 split a+b (MF16, MF18)** — Due sotto-scenari separati:
   - **CDU-01a Operatore:** accesso via RUPAR/IRIDE/SPID; selezione profilo operatore visibile
   - **CDU-01b Cittadino:** accesso via SPID/CIE su webapp separata; nessuna profilazione iniziale; pulsante "Deleghe" per agire come delegato
   Precondizioni separate per scenario.

5. **Deleghe (MF20, MF22)** — Pulsante "Deleghe" → elenco deleganti attivi (recuperati via servizio Gestione Deleghe) → selezione carica cruscotto consensi del delegante. Scenario già attivo in produzione (nota implementativa).

### Tema B — Ciclo vita consenso

**Decisioni chiave (MF11, MF14):** il diagramma stati attuale è incompleto. Mancano transizioni dirette ATTIVO↔NEGATO senza richiedere nuova informativa, quando il cittadino o un operatore modifica il valore. Aggiungere entrambe le frecce.

### Tema C — CDU-02/03/04/05 dinamica cittadino

**Decisioni chiave:**

- CDU-02 (MF26): tutti i tipi configurati, regionale=1, aziendale=N per azienda — anche quelli già espressi
- Informativa per ente (MF28): visualizzazione singola per consenso aziendale con informativa specifica per azienda
- Pagina dinamica (MF30, MF35): config DB → UI rendering
- Pulsante unico Salva (MF37): UI non distingue rilascio da modifica
- Modifica = stesse regole (MF39, MF41, MF43)
- CDU-04/05 merge cittadino (MF45): CDU-05 separato solo per Operatore

**Concept dedicato:** [[wiki/concepts/composizione-dinamica-form-consenso\|Composizione Dinamica Form Consenso — Single Source of Truth]]

### Tema D — CDU-06 PDF

**Decisioni chiave (MF47, MF49, MF51):**
- CDU-06 ha scope **ridotto**: il cittadino può stampare in PDF **solo l'informativa accettata**, senza:
  - firma digitale
  - valore consenso espresso
- La struttura completa del PDF (con dettagli aggiuntivi) era una **proposta implementativa**: tutto marcato `[PROPOSTA]` da concordare con CSI.

Implicazione: il CDU non genera "attestazione completa del consenso", solo "copia dell'informativa accettata".

### Tema E — CDU-07 AURA / no SistemaTS

**Decisioni chiave (MF53, MF55):**
- AURA invocata con **FindProfiliAnagrafici** + **getProfiloSanitario**
- Se il CF non viene trovato: messaggio fisso "La ricerca con il filtro fornito non ha prodotto risultati"
- **SistemaTS NON viene chiamato** — né come fallback né come supporto. Eliminare ogni riferimento a SistemaTS dal §6.7.

Wiki: rimuovere "Fallback SistemaTS" da [[wiki/concepts/sistemi-esterni-integrati\|Sistemi Esterni Integrati]] AURA. Aggiornare [[wiki/analyses/analysis-gap-as-is-to-be\|Analisi Gap AS-IS → TO-BE — Gestione Consensi]].

### Tema F — Operatore composizione dinamica SSoT

**Decisione chiave (MF57R56):** vincolo di unicità UI fra Cittadino e Operatore. Form Renderer unico. CDU-09/10/11 devono avere paragrafo "Composizione dinamica pagina operatore" che rimanda esplicitamente a §6.2 e §6.4 lato cittadino.

Differenze ammesse solo a livello metadati di tracciatura sorgente (`fonte_id`, `login_operazione`, `ruoloop_id`).

Architetturalmente: aggiungere componente "Form Renderer dinamico" in SRS §3.3.

**Concept dedicato:** [[wiki/concepts/composizione-dinamica-form-consenso\|Composizione Dinamica Form Consenso — Single Source of Truth]]

### Tema G — Notifica cittadino post-COMPLETATO

**Decisione chiave (MF33R31):**
- Canale: **Notificatore di Deleghe** (NON UNP — User Notification Platform usata per altri canali)
- Timing: notifica al cittadino/delegato parte **SOLO dopo** che le notifiche alle aziende risultano `COMPLETATO`

Implicazione: dipendenza temporale stretta fra BATCH-01 (notifica aziende) e notifica cittadino. Se BATCH-01 fallisce/ritarda, anche la notifica cittadino è ritardata.

Wiki: distinguere chiaramente **Notificatore di Deleghe** vs **UNP (User Notification Platform)** in [[wiki/concepts/sistemi-esterni-integrati\|Sistemi Esterni Integrati]] — sono due servizi diversi.

### Tema H — Batch SQL/scheduling

**Decisioni chiave:**

- **MF24R23:** logica selezione endpoint riferita a §6.14 CDU-14 e ALG01. Cross-reference esplicito.
- **MF64R63:** tabella transizioni stato per `cons_t_notifica` e `cons_t_batch` da aggiungere. Scheduling BATCH-01: **5 minuti** (era AS-IS 30 min) con `SELECT FOR UPDATE SKIP LOCKED` su `cons_t_notifica` per evitare sovrapposizioni fra esecuzioni successive.
- **MF66R65:** SQL canonico per BATCH-02 per determinare `nuova_d_informativa_id` (informativa corrente non scaduta):
  ```sql
  SELECT d_informativa_id AS nuova_d_informativa_id
  FROM cons_d_informativa
  WHERE sotto_tipo_consenso = :sotto_tipo_consenso_scaduta
    AND (data_scadenza IS NULL OR data_scadenza > NOW())
    AND data_decorrenza <= NOW()
    AND data_cancellazione IS NULL
  ORDER BY data_decorrenza DESC
  LIMIT 1
  ```

### Tema I — Già consolidato (ex TR30, ex TR34)

- **TR58 = TR30** (MF59-62R58): security CDU-15/16. Wiki: [[wiki/concepts/sicurezza-cdu-15-16\|Sicurezza CDU-15-16 — Modello Autorizzazione per Ente]]. Aggiornare cross-ref ID.
- **TR68 = TR34** (MF69R68): alternativa BATCH-03 PULL. Wiki: [[wiki/concepts/alternativa-batch-03-pull\|Alternativa BATCH-03 — PULL CDU-17 (centro stella)]]. Aggiornare cross-ref ID.

### Tema J — Aperto

- **SC67:** "da approfondire e verificare meglio" su INSERT cons_t_consenso da batch storicizzazione. Da discutere prima di chiusura SRS.

---

## Propagazione wiki — riepilogo file impattati

| File | Modifica | Stato |
|---|---|---|
| `wiki/concepts/composizione-dinamica-form-consenso.md` | Nuovo concept SSoT | ✅ creato |
| `wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3.md` | Questo file | ✅ creato |
| `wiki/concepts/ciclo-vita-consenso.md` | Transizioni ATTIVO↔NEGATO + UI termini | ✅ |
| `wiki/concepts/gestione-consensi-applicativo.md` | Profili, CDU-01 split, deleghe, CDU-04/05 merge, CDU-06 scope, LIS | ✅ |
| `wiki/concepts/batch-processes.md` | BATCH-01 5min+SKIP LOCKED, BATCH-02 SQL, Notificatore Deleghe | ✅ |
| `wiki/concepts/sistemi-esterni-integrati.md` | AURA dettaglio + no SistemaTS, Notificatore Deleghe, LIS canale | ✅ |
| `wiki/concepts/sicurezza-cdu-15-16.md` | Cross-ref ID TR58 (ex TR30) | ✅ |
| `wiki/concepts/alternativa-batch-03-pull.md` | Cross-ref ID TR68 (ex TR34) | ✅ |
| `wiki/analyses/analysis-gap-as-is-to-be.md` | Scheduling, SistemaTS, UI, CDU-01 split, CDU-04/05 merge, LIS | ✅ |
| `wiki/sources/2026-03-02-conspref-srs-v1-revised.md` | Postilla v3_lavorazione | ✅ |
| `wiki/index.md`, `wiki/log.md`, `wiki/overview.md` | Aggiornamento indice e log | ✅ |

---

## ADR correlati

Le risposte MF hanno generato decisioni architetturali formalizzate come ADR. Mapping MF → ADR:

| Tema MF | ADR generati |
|---|---|
| A — Canali (MF3/MF4) | [ADR-017](ADR-017-lis-terzo-canale.md) LIS terzo canale — ⚠️ superato da [ADR-020](ADR-020-lis-integrazione-be-esistente.md) (06/08/2026) |
| A — CDU-01 split (MF16/MF18) | [ADR-010](ADR-010-cdu-01-split.md) Split CDU-01 |
| B — Stati (MF11/MF14) | [ADR-011](ADR-011-merge-cdu-04-05-cittadino.md) transizioni dirette in merge CDU-04/05 |
| C — Composizione dinamica (MF26/28/30/35/37/39/41/43) | [ADR-008](ADR-008-ssot-form-renderer.md) SSoT Form Renderer |
| C — Merge CDU-04/05 cittadino (MF45) | [ADR-011](ADR-011-merge-cdu-04-05-cittadino.md) — ⚠️ superato da [ADR-021](ADR-021-perimetro-solo-operatore.md) (06/08/2026) |
| D — CDU-06 PDF (MF47/49/51) | [ADR-019](ADR-019-cdu-06-pdf-scope-ridotto.md) CDU-06 scope ridotto — ⚠️ superato da [ADR-021](ADR-021-perimetro-solo-operatore.md) (06/08/2026) |
| E — No SistemaTS (MF53/55) | [ADR-009](ADR-009-eliminazione-sistemats.md) Eliminazione SistemaTS |
| F — SSoT operatore (MF57) | [ADR-008](ADR-008-ssot-form-renderer.md) |
| G — Notificatore Deleghe (MF33) | [ADR-012](ADR-012-notificatore-deleghe-post-completato.md) |
| H — Scheduling (MF64) | [ADR-007](ADR-007-batch-01-5min-skip-locked.md) BATCH-01 5min |
| H — BATCH-02 SQL (MF66) | [ADR-016](ADR-016-scaduto-async-batch-02.md) SCADUTO async |
| I — Sicurezza (MF59-62, ex TR30/TR58) | [ADR-005](ADR-005-sicurezza-cdu-15-16.md) + [ADR-004](ADR-004-no-api-gateway.md) |
| I — BATCH-03 PULL (MF69, ex TR34/TR68) | [ADR-006](ADR-006-batch-03-pull-cdu-17.md) — **accepted** (confermato 20/07/2026) |
