---
{"dg-publish":true,"permalink":"/wiki/docs/adr/adr-017-lis-terzo-canale/","title":"LIS come terzo canale di acquisizione consenso","tags":["lis","canali-acquisizione","mf3","mf4","integrazione","diagramma-contesto"],"dg-note-properties":{"adr":17,"title":"LIS come terzo canale di acquisizione consenso","status":"superseded","date":"2026-05-14","updated":"2026-08-06","deciders":["Marco Forneris","Exprivia"],"supersedes":[],"superseded-by":[20],"tags":["lis","canali-acquisizione","mf3","mf4","integrazione","diagramma-contesto"],"related_wiki":["[[Gestione Consensi - Applicativo]]","[[Sistemi Esterni Integrati]]","[[composizione-dinamica-form-consenso|Composizione Dinamica Form Consenso]]"],"sources":["[[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|Risposte MF SRS v3]] MF3R1, MF4R1"]}}
---


# ADR-017: LIS terzo canale di acquisizione

## Status

`superseded by [[wiki/docs/adr/ADR-020-lis-integrazione-be-esistente\|ADR-020]]` — decisione originaria MF3R1 + MF4R1 sulla revisione SRS v3.

> ⚠️ **Superata (call CSI 06/08/2026, INT-03 chiuso):** CSI ha chiarito che **non esiste un terzo canale di acquisizione consensi**. Per i nuovi sviluppi (funzionali e non funzionali), l'acquisizione presso LIS/RIS **avviene tramite un'integrazione BE già presente nel codice sorgente AS-IS** — non un nuovo canale UI da costruire. Attività residua: verificare l'integrazione esistente e migrarla al nuovo stack. Vedi [[wiki/docs/adr/ADR-020-lis-integrazione-be-esistente\|ADR-020]] per la decisione aggiornata. Questa pagina resta come registro storico della decisione originaria (poi rivelatasi basata su un'interpretazione errata di "canale").

## Context

Bozza SRS v3 documentava due canali di acquisizione consensi:
1. **Webapp Cittadino** (SPID/CIE)
2. **Webapp Operatore** (PUA — operatore opera per conto dell'assistito)

CSI ha confermato l'esistenza di un terzo canale operativo:
> "Consensi esprimibili anche presso LIS oltre webapp cittadino e Operatore"

LIS è una funzione di front-office laboratorio (Laboratorio?). L'acronimo non è definito formalmente nella documentazione corrente — punto aperto INT-03.

## Decision

Aggiungere **LIS** come terzo canale di acquisizione consensi:

| Canale           | Descrizione                                       | Profilo accesso             |
| ---------------- | ------------------------------------------------- | --------------------------- |
| Webapp Cittadino | Webapp dedicata SPID/CIE                          | Cittadino autenticato       |
| Webapp Operatore | Operatore opera per conto dell'assistito          | Operatore PUA (RUPAR/IRIDE) |
| **LIS**          | Acquisizione presso il Laboratorio (front-office) | Operatore LIS dedicato      |

Implicazioni documentali:
- SRS §1/§2 diagramma di contesto: 3 canali, non 2
- Didascalia diagramma: "Consensi esprimibili anche presso LIS oltre webapp cittadino e Operatore" (MF4R1)
- Tracciatura: `fonte_id` con valore distintivo per LIS (es. `LIS`)

Composizione dinamica del form ([[wiki/docs/adr/ADR-008-ssot-form-renderer\|ADR-008-ssot-form-renderer]]) si applica anche al canale LIS: il Form Renderer può essere riusato in modalità operatore LIS.

## Consequences

### Positive
- Documentazione SRS allineata alla realtà operativa
- Tracciatura `fonte_id` distingue acquisizioni LIS da webapp citt e operatore PUA
- Architettura SSoT Form Renderer ([[wiki/docs/adr/ADR-008-ssot-form-renderer\|ADR-008-ssot-form-renderer]]) gestisce il canale senza divergenze

### Negative
- **Acronimo LIS non definito** ufficialmente — INT-03 aperto
- Specifica integrazione canale LIS **da chiarire con CSI** (autenticazione operatore LIS, eventuale endpoint dedicato, modalità tracciatura)
- Più scenari di test (3 canali × CDU rilevanti)

### Neutral
- Nessun nuovo CDU dedicato — LIS riusa pattern Operatore con `fonte_id` distinto

## Open issues

- INT-03: acronimo LIS formalizzato + spec integrazione canale acquisizione (tracker [[wiki/analyses/analysis-2026-05-14-punti-aperti-csi\|Punti Aperti CSI]] §6)

## Alternatives considered

| Alternativa | Motivo scarto |
|---|---|
| Trattare LIS come variante Operatore senza canale dedicato | Cliente lo nomina come canale distinto (MF4R1); tracciatura `fonte_id` richiede valore proprio |
| Webapp LIS dedicata | Eccessivo finché spec integrazione non chiarita |

## References

- [[wiki/concepts/gestione-consensi-applicativo\|Gestione Consensi - Applicativo]] §Canali di acquisizione
- [[wiki/concepts/sistemi-esterni-integrati\|Sistemi Esterni Integrati]] §LIS
- [[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|Risposte MF SRS v3]] tema A (MF3R1, MF4R1)
- Correlato: [[wiki/docs/adr/ADR-008-ssot-form-renderer\|ADR-008-ssot-form-renderer]] SSoT Form Renderer (riuso per LIS), [[wiki/docs/adr/ADR-011-merge-cdu-04-05-cittadino\|ADR-011-merge-cdu-04-05-cittadino]] merge CDU-04/05 (UX cittadino, non LIS)
