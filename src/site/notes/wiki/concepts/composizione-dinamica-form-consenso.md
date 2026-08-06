---
{"dg-publish":true,"permalink":"/wiki/concepts/composizione-dinamica-form-consenso/","title":"Composizione Dinamica Form Consenso — Single Source of Truth","tags":["ui","form","dinamico","cdu-02","cdu-03","cdu-04","cdu-09","cdu-10","cdu-11","ssot","form-renderer","mf57","mf26","mf30"],"dg-note-properties":{"title":"Composizione Dinamica Form Consenso — Single Source of Truth","aliases":["Composizione Dinamica Form Consenso — Single Source of Truth"],"type":"concept","tags":["ui","form","dinamico","cdu-02","cdu-03","cdu-04","cdu-09","cdu-10","cdu-11","ssot","form-renderer","mf57","mf26","mf30"],"created":"2026-05-14","updated":"2026-05-14","sources":["2026-03-02-conspref-srs-v1-revised"],"related":["[[Gestione Consensi - Applicativo]]","[[wiki/concepts/ciclo-vita-consenso\|Ciclo di Vita del Consenso]]","[[Sistemi Esterni Integrati]]","[[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|analysis-2026-05-14-risposte-mf-srs-v3]]"]}}
---


# Composizione Dinamica Form Consenso — Single Source of Truth

**Origine:** risposte MF a revisione SRS v3 — commenti MB29, MB27, MB34, MB36, MB38, MB40, MB42, MB56 (vedi [[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|analysis-2026-05-14-risposte-mf-srs-v3]] tema C e tema F).

> 🔴 **Perimetro progetto (call CSI 06/08/2026, [[wiki/docs/adr/ADR-021-perimetro-solo-operatore\|ADR-021]]):** questo progetto sviluppa **solo la Webapp Operatore**. Il pattern SSoT sotto descritto nasce per un riuso Cittadino+Operatore, ma in questo progetto **si applica solo lato Operatore** (§4). Le sezioni su CDU-02/03/04 cittadino (§3) restano come contesto storico — la Webapp Cittadino non è un deliverable di questo progetto.

Pattern architetturale per il rendering delle pagine di consenso lato **Operatore** (CDU-09, CDU-10, CDU-11 — ✅ in scope). Nato come pattern unico condiviso anche con **Cittadino** (CDU-02, CDU-03, CDU-04 — ❌ fuori scope di sviluppo, vedi banner sopra). Vincolo originario: nessuna divergenza di struttura fra le due interfacce — resta valido come principio, ma solo la Webapp Operatore è costruita da questo progetto.

---

## 1. Vincolo architetturale (MF57R56)

**Single Source of Truth (SSoT):** la stessa logica di rendering della pagina cittadino viene riusata lato operatore. Non sono ammesse divergenze nella struttura dei campi tra interfaccia Cittadino e interfaccia Operatore per lo stesso `sotto_tipo_consenso`.

**Conseguenza implementativa:** motore di rendering **"Form Renderer dinamico"**, in questo progetto usato dalla **sola** Webapp Operatore (vedi [[wiki/docs/adr/ADR-021-perimetro-solo-operatore\|ADR-021]]). Se la Webapp Cittadino esistente riuserà lo stesso componente non è verifica/garanzia di questo progetto. Da riflettere nel SRS §3.3 Componenti software come componente identificato.

---

## 2. Regole di composizione (MF26, MF28, MF30, MF35)

La pagina di consenso si compone **dinamicamente** sulla base della configurazione del tipo di consenso. Nulla è hardcoded a livello UI: tutto deriva dalla configurazione DB.

### 2.1 Cosa è dinamico

| Elemento UI                          | Sorgente DB                                  | Note                                     |
| ------------------------------------ | -------------------------------------------- | ---------------------------------------- |
| Campi visualizzati                   | `cons_d_sotto_tipo_cons` + tabelle correlate | Letti per `sotto_tipo_consenso`          |
| Valori ammessi per `valore_consenso` | `cons_d_sotto_tipo_cons` / tabelle valori    | Non solo SI/NO — possono essere multipli |
| Domande/flag opzionali               | Tabelle configurazione                       | Mostrate solo se previste                |
| Flag "campo richiesto"               | Config tipo consenso                         | Vincolo validazione                      |
| Layout informativa                   | Config aziendale vs regionale                | Vedi §2.3                                |

### 2.2 Lista consensi mostrati (CDU-02 MF26)

> Il sistema mostra **tutti** i tipi di consenso configurati per il cittadino, indipendentemente dallo stato espresso.

| Tipologia | Visualizzazione |
|---|---|
| Regionale | **Un singolo record** con stato globale (la scelta si propaga a tutte le ASR) |
| Aziendale | **Un record per ogni azienda** con il relativo stato (inclusi quelli già espressi) |

Errore da evitare: mostrare solo i consensi non ancora espressi. Il cittadino deve vedere anche quelli già espressi per poterli modificare.

### 2.3 Informativa per consenso aziendale (MF28R27)

> Per i consensi aziendali, se ogni azienda ha una propria informativa specifica, il sistema deve permettere la visualizzazione dell'informativa della singola azienda **prima** di raccogliere il consenso per quell'azienda.

Implementazione: dropdown o tab per azienda; alla selezione, fetch dell'informativa specifica via `d_informativa_id` legato all'ente.

---

## 3. Pulsante unico "Salva" lato cittadino (MF37R36) ❌ OUT scope

> 🔴 **Fuori dal perimetro di sviluppo** (call CSI 06/08/2026, [[wiki/docs/adr/ADR-021-perimetro-solo-operatore\|ADR-021]]) — decisione [[wiki/docs/adr/ADR-011-merge-cdu-04-05-cittadino\|ADR-011]] superseded. Sezione mantenuta come contesto storico.

> Web app cittadino: pulsante unico **«Salva»**. Operazioni tecniche distinte (rilascio CDU-03 vs modifica CDU-04) gestite internamente, non esposte UI.

Razionale (MB38, MB44): il cittadino non distingue "rilascio" da "modifica" — vuole solo salvare la propria scelta. Distinzione CDU-03/CDU-04 rilevante solo per logica backend e tracciatura.

**Conseguenza (MF45R44):** lato webapp Cittadino, CDU-05 **NON è caso d'uso separato** — flusso inglobato in CDU-04. La distinzione CDU-04/CDU-05 (modifica con scadenza vs cambio valore) resta rilevante solo per:
- l'Operatore (UI distinta per casistiche operative diverse)
- la logica interna (riaccettazione informativa o no)

---

## 4. Lato Operatore — differenze (MF57R56)

Layout dinamico **identico** al lato cittadino. Le sole differenze riguardano i metadati di tracciatura sorgente:

| Campo | Lato Cittadino | Lato Operatore |
|---|---|---|
| `fonte_id` | derivato da contesto webapp citt. (es. `WEB_CITT`) | `'PASS'` |
| `login_operazione` | CF cittadino autenticato | login operatore PUA |
| `ruoloop_id` | NULL | ID ruolo operatore PUA |

> ⚠️ **Nota (ADR-020, 06/08/2026):** LIS **non** passa dal Form Renderer né dai canali Cittadino/Operatore — è un'integrazione BE esistente nel sorgente AS-IS ([[wiki/concepts/sistemi-esterni-integrati\|Sistemi Esterni Integrati]] §LIS), da verificare e migrare al nuovo stack. Un eventuale `fonte_id='LIS'` è un valore AS-IS da preservare in migrazione, non un contesto webapp del Form Renderer SSoT.

I CDU operatore coinvolti:
- **CDU-09** Rilascio consenso per conto dell'assistito
- **CDU-10** Modifica consenso per conto dell'assistito
- **CDU-11** Cambio valore consenso per conto dell'assistito

Ognuno deve avere paragrafo "Composizione dinamica pagina operatore" che rimanda esplicitamente alle regole §6.2 (CDU-02 cittadino) e §6.4 (CDU-04 cittadino) come unica fonte autorevole.

---

## 5. Implementazione raccomandata

### 5.1 Componente architetturale SRS §3.3
Aggiungere componente **"Form Renderer dinamico"** nei componenti software:
- Tecnologia: Angular component library condivisa
- Input: `tipo_consenso`, `sotto_tipo_consenso`, `codice_ente` (per aziendali), `modalita` (`citt` | `operatore`)
- Output: form renderizzato + binding bidirezionale con backend
- Riuso: stesso componente importato da `app-cittadino` e `app-operatore`

### 5.2 Schema configurazione DB
Tabella principale: `cons_d_sotto_tipo_cons` con campi configurativi:
- `valori_ammessi_json` o tabella correlata `cons_d_valore_consenso`
- `domande_opzionali_json` o tabella correlata
- `flag_informativa_per_ente` (boolean)
- `flag_richiesto` (boolean per campo)

### 5.3 Pattern fetch
```
GET /api/v1/config/sotto-tipo-consenso/{sotto_tipo}
  ↓
{
  "sotto_tipo": "ROL",
  "campi": [...],
  "valori_ammessi": [...],
  "domande": [...],
  "informativa_per_ente": true
}
```

Il backend (Spring Boot 3) espone l'endpoint, il Form Renderer Angular lo consuma alla `init` della pagina consenso. Cache lato browser opportuna.

---

## 6. Mapping commenti MF coperti

| Commento revisore | Risposta MF | Sezione di questo concept |
|---|---|---|
| MB29 (composizione dinamica) | MF30R29 | §2 |
| MB27 (informativa per azienda) | MF28R27 | §2.3 |
| MB34 (valori dinamici) | MF35R34 | §2.1 |
| MB36 (manca compilazione dinamica) | MF37R36 | §3 |
| MB38 (pulsante unico) | MF37R36 | §3 |
| MB40/MB42 (modifica dinamica) | MF39/41/43 | §2 + §3 |
| MB44 (CDU-04/05 merge cittadino) | MF45R44 | §3 |
| MB56 (logica operatore = cittadino) | MF57R56 | §1 + §4 |
| SC25 (lista completa CDU-02) | MF26R25 | §2.2 |

---

## 7. Gap SRS da chiudere

> ⚠️ Nessuna modifica SRS in questa fase (in attesa di conferma utente). Colonna Scope aggiunta 06/08/2026 per riflettere [[wiki/docs/adr/ADR-021-perimetro-solo-operatore\|ADR-021]].

| # | Sezione SRS | Modifica | Scope |
|---|---|---|---|
| G1 | §6.2 CDU-02 | Riscrittura punto "tipi visibili" come da MF26R25 (regionale=1, aziendale=N per azienda, tutti i tipi mostrati) | ❌ OUT (Cittadino) |
| G2 | §6.3 CDU-03 | Pulsante unico Salva (MF37) — niente UI rilascio vs modifica | ❌ OUT (Cittadino) |
| G3 | §6.4 CDU-04 | Stesse regole composizione di §6.2 — riferimento esplicito | ❌ OUT (Cittadino) |
| G4 | §6.5 CDU-05 | Lato cittadino: inglobato in CDU-04 (fuori scope). Lato operatore: caso d'uso separato resta | ✅ IN (solo parte Operatore) |
| G5 | §6.9, §6.10, §6.11 | Aggiungere paragrafo "Composizione dinamica pagina operatore" per ogni CDU — rimanda alle regole di composizione (non più a §6.2/§6.4 cittadino, fuori scope) | ✅ IN |
| G6 | §3.3 Componenti software | Aggiungere "Form Renderer dinamico" come componente — in questo progetto usato dalla sola Webapp Operatore | ✅ IN |

---

## 8. Riferimenti

- Dettaglio risposte MF: [[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|analysis-2026-05-14-risposte-mf-srs-v3]]
- Profili utente: [[wiki/concepts/gestione-consensi-applicativo\|Gestione Consensi - Applicativo]]
- Stati consenso (rendering dipendente): [[wiki/concepts/ciclo-vita-consenso\|Ciclo di Vita del Consenso]]
- AURA (popolamento dati assistito lato operatore): [[wiki/concepts/sistemi-esterni-integrati\|Sistemi Esterni Integrati]]

---

## ADR correlati

| ADR | Decisione |
|---|---|
| [ADR-008](ADR-008-ssot-form-renderer.md) | SSoT Form Renderer unico (questa concept è la fonte autoritativa) — nota scope: solo Webapp Operatore |
| [ADR-011](ADR-011-merge-cdu-04-05-cittadino.md) | Merge CDU-04/05 lato Cittadino (pulsante unico Salva) — **superseded** da ADR-021 |
| [ADR-021](ADR-021-perimetro-solo-operatore.md) | Perimetro progetto: solo Webapp Operatore |
