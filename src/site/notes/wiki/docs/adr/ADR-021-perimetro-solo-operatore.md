---
{"dg-publish":true,"permalink":"/wiki/docs/adr/adr-021-perimetro-solo-operatore/","title":"Perimetro progetto ridotto a Webapp Operatore — Webapp Cittadino esclusa","tags":["perimetro","scope","webapp-operatore","webapp-cittadino"],"dg-note-properties":{"adr":21,"title":"Perimetro progetto ridotto a Webapp Operatore — Webapp Cittadino esclusa","status":"accepted","date":"2026-08-06","deciders":["CSI Piemonte","Exprivia"],"supersedes":[11,19],"superseded-by":[],"tags":["perimetro","scope","webapp-operatore","webapp-cittadino"],"related_wiki":["[[Gestione Consensi - Applicativo]]","[[GASP Salute]]","[[Composizione Dinamica Form Consenso]]","[[wiki/docs/adr/ADR-010-cdu-01-split\|ADR-010-cdu-01-split]]","[[wiki/docs/adr/ADR-008-ssot-form-renderer\|ADR-008-ssot-form-renderer]]"],"sources":["Call CSI 06/08/2026"]}}
---


# ADR-021: Perimetro progetto ridotto a Webapp Operatore — Webapp Cittadino esclusa

## Status

`accepted` — chiarimento CSI in call 06/08/2026. Supersede [[wiki/docs/adr/ADR-011-merge-cdu-04-05-cittadino\|ADR-011]] e [[wiki/docs/adr/ADR-019-cdu-06-pdf-scope-ridotto\|ADR-019]].

## Context

Finora la wiki e l'SRS in lavorazione hanno modellato il sistema TO-BE come **due webapp** equivalenti in scope di sviluppo:
- **Webapp Cittadino** (SPID/CIE via [[wiki/concepts/gasp-salute\|GASP Salute]]) — CDU-01b, CDU-02, CDU-03, CDU-04, CDU-06
- **Webapp Operatore** (PUA/RUPAR/IRIDE) — CDU-01a, CDU-05, CDU-07÷CDU-14

Su questa base erano state prese decisioni architetturali che assumevano entrambe le webapp come deliverable di questo progetto: split CDU-01a/01b ([[wiki/docs/adr/ADR-010-cdu-01-split\|ADR-010]]), SSoT Form Renderer condiviso Citt+Op ([[wiki/docs/adr/ADR-008-ssot-form-renderer\|ADR-008]]), UX cittadino semplificata con pulsante unico ([[wiki/docs/adr/ADR-011-merge-cdu-04-05-cittadino\|ADR-011]]), download PDF informativa lato cittadino ([[wiki/docs/adr/ADR-019-cdu-06-pdf-scope-ridotto\|ADR-019]]).

In call CSI del 06/08/2026, il cliente ha chiarito che **i progetti in gestione non sono due ma uno solo**: la **Webapp Operatore**. La Webapp Cittadino **esiste** ma **non è un deliverable di questo progetto** — resta fuori dalla gestione di questi nuovi sviluppi (chi la gestisce, se e quando verrà aggiornata, non è di nostra competenza in questo engagement).

## Decision

- **In scope di sviluppo:** Webapp Operatore — CDU-01a, CDU-05, CDU-07÷CDU-14 (area Operatore + Back Office), CDU-15÷CDU-17 (API per SIA, machine-to-machine).
- **Fuori scope di sviluppo:** Webapp Cittadino — CDU-01b, CDU-02, CDU-03, CDU-04, CDU-06. La webapp esiste ma non viene costruita/aggiornata da questo progetto.
- **[[wiki/concepts/gasp-salute\|GASP Salute]] fuori scope tecnico:** era l'IdP per l'accesso diretto del cittadino (SPID/CIE); senza CDU-01b da costruire, non c'è integrazione GASP da progettare in questo progetto.
- **[[wiki/docs/adr/ADR-011-merge-cdu-04-05-cittadino\|ADR-011]] e [[wiki/docs/adr/ADR-019-cdu-06-pdf-scope-ridotto\|ADR-019]] superseded:** trattano esclusivamente funzionalità cittadino (UX pulsante unico, download PDF) — interamente fuori dal perimetro di questo progetto. Restano come registro storico della decisione originaria.
- **[[wiki/docs/adr/ADR-010-cdu-01-split\|ADR-010]] resta `accepted`, non superseded:** la distinzione tecnica CDU-01a/01b resta corretta concettualmente; annotato che solo CDU-01a è in scope di sviluppo.
- **[[wiki/docs/adr/ADR-008-ssot-form-renderer\|ADR-008]] resta `accepted`, non superseded:** il pattern Form Renderer resta valido per la Webapp Operatore; la motivazione "riuso Citt+Op" non è più applicabile in questo progetto — se la Webapp Cittadino esistente userà lo stesso renderer non è verifica/garanzia di questo progetto.

## Consequences

### Positive
- Perimetro di sviluppo chiaro: 1 webapp, non 2 — riduce ambiguità su cosa va costruito
- Elimina lavoro di progettazione non necessario: integrazione GASP/SAML2, UX cittadino dedicata, download PDF cittadino
- CDU-15÷17 (API SIA) e area Operatore/Back Office restano interamente validi e prioritari

### Negative
- Documentazione SRS §1/§2 (diagramma di contesto), §3 (profili), catalogo CDU **da correggere** per riflettere il perimetro ridotto — **allineamento SRS rimandato**, da eseguire solo dopo conferma esplicita dell'utente
- Decisioni pregresse (ADR-010, ADR-008) restano tecnicamente valide ma la loro applicabilità pratica in questo progetto si riduce (solo metà del loro contesto originario è ancora costruito da noi)
- Rischio di confusione se la Webapp Cittadino esistente diverge nel tempo dal modello dati/API che il backend Operatore/SIA implementa — non è nostro compito verificarlo, ma va tenuto presente

### Neutral
- Nessun impatto sul modello dati backend (`cons_t_consenso`, storicizzazione, batch) — channel-agnostic, resta valido per entrambe le webapp indipendentemente da chi le costruisce

## Alternatives considered

| Alternativa | Motivo scarto |
|---|---|
| Mantenere entrambe le webapp in scope, aspettando ulteriore conferma | Contraddetto esplicitamente da CSI in call 06/08/2026 — il perimetro è chiaro, non serve attendere |
| Cancellare interamente ADR-010/ADR-008 | Errato: restano tecnicamente corretti per la parte Operatore; solo la parte "riuso con Citt" perde rilevanza pratica |

## Open issues

- Correggere SRS §1/§2/§3 e catalogo CDU per riflettere il perimetro (solo dopo conferma utente)
- Verificare se il modello dati/API esposto dal backend Operatore/SIA deve restare compatibile con la Webapp Cittadino esistente (proprietà/contratto non chiarito in questa call)

## References

- [[wiki/concepts/gestione-consensi-applicativo\|Gestione Consensi - Applicativo]] §Canali di acquisizione, §Profili utente
- [[wiki/concepts/gasp-salute\|GASP Salute]] (fuori scope tecnico)
- [[wiki/docs/adr/ADR-010-cdu-01-split\|ADR-010-cdu-01-split]] (resta accepted, nota di scope)
- [[wiki/docs/adr/ADR-008-ssot-form-renderer\|ADR-008-ssot-form-renderer]] (resta accepted, nota di scope)
- [[wiki/docs/adr/ADR-011-merge-cdu-04-05-cittadino\|ADR-011-merge-cdu-04-05-cittadino]], [[wiki/docs/adr/ADR-019-cdu-06-pdf-scope-ridotto\|ADR-019-cdu-06-pdf-scope-ridotto]] — superseded da questo ADR
