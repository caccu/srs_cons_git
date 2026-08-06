---
{"dg-publish":true,"permalink":"/wiki/docs/adr/adr-019-cdu-06-pdf-scope-ridotto/","title":"CDU-06 PDF — scope ridotto a sola informativa accettata, no firma, no valore consenso","tags":["cdu-06","pdf","informativa","mf47","scope","ux-cittadino"],"dg-note-properties":{"adr":19,"title":"CDU-06 PDF — scope ridotto a sola informativa accettata, no firma, no valore consenso","status":"superseded","date":"2026-05-14","updated":"2026-08-06","deciders":["Marco Forneris","Exprivia"],"supersedes":[],"superseded-by":[21],"tags":["cdu-06","pdf","informativa","mf47","scope","ux-cittadino"],"related_wiki":["[[Gestione Consensi - Applicativo]]","[[wiki/concepts/informativa\|Informativa Consenso]]","[[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|Risposte MF SRS v3]]"],"sources":["[[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|Risposte MF SRS v3]] MF47R46, MF49R48, MF51R50"]}}
---


# ADR-019: CDU-06 PDF scope ridotto

## Status

`superseded by [[wiki/docs/adr/ADR-021-perimetro-solo-operatore\|ADR-021]]` — decisione originaria MF47R46 sulla revisione SRS v3.

> ⚠️ **Superata (call CSI 06/08/2026):** CDU-06 è un caso d'uso esclusivamente cittadino, e la Webapp Cittadino è **fuori dal perimetro di sviluppo** di questo progetto (vedi [[wiki/docs/adr/ADR-021-perimetro-solo-operatore\|ADR-021]]). Questa decisione resta come registro storico ma non guida più lo sviluppo.

## Context

Bozza SRS v3 prevedeva CDU-06 come "Download PDF attestazione consenso" con struttura ricca:
- Informativa accettata (testo)
- Valore consenso espresso (SI/NO)
- Firma digitale (eIDAS o equivalente)

Discussione cliente:
- **Firma eIDAS non richiesta** (Q&A CSI #13)
- L'attestazione del valore consenso espresso non è il valore informativo principale per il cittadino — il vero scopo del download è avere copia dell'informativa accettata, per archivio personale
- Struttura ricca complica scope e tempi (firma digitale = integrazione esterna)

## Decision

CDU-06 ha **scope ridotto**:

- Cittadino stampa PDF della **sola informativa accettata**
- **Senza** firma digitale
- **Senza** valore consenso espresso
- Struttura ulteriore (eventuali sezioni aggiuntive, layout) → marcata `[PROPOSTA]` da concordare con CSI in iterazione successiva

Lato Operatore: nessun caso d'uso analogo nel TO-BE (l'operatore non scarica PDF per conto dell'assistito).

## Consequences

### Positive
- Scope chiaro e implementabile in fase 1
- Niente integrazione firma digitale → niente dipendenza esterna su servizi eIDAS
- Niente disclosure valore consenso in PDF → riduce rischio data leak in caso di stampa condivisa
- Allineamento con UX cittadino [[wiki/docs/adr/ADR-011-merge-cdu-04-05-cittadino\|ADR-011-merge-cdu-04-05-cittadino]] (pulsante unico Salva, no esposizione tecnicalità)

### Negative
- "Attestazione consenso completo" (valore + informativa + firma) **non disponibile**: cittadino vuole prova legale → soluzione esterna (es. richiesta presso ASR)
- Eventuale richiesta futura "PDF attestazione completo" → nuovo CDU, non extension di CDU-06

### Neutral
- Generazione PDF lato backend Spring Boot (libreria standard, es. Apache PDFBox o iText)
- Struttura specifica `[PROPOSTA]` da iterare con CSI

## Alternatives considered

| Alternativa | Motivo scarto |
|---|---|
| PDF completo con firma eIDAS | Q&A CSI #13: firma non richiesta; complica scope sprint 1 |
| PDF con valore consenso ma no firma | Esposizione valore in stampa non desiderata; cliente ha esplicitamente escluso (MF47R46) |
| Niente CDU-06 | Cliente richiede stampa informativa per archivio cittadino |

## Open issues

- GOV-02 (tracker [[wiki/analyses/analysis-2026-05-14-punti-aperti-csi\|Punti Aperti CSI]] §8): validazione `[PROPOSTA]` struttura PDF (MF49R48, MF51R50) prima di Sprint 2

## References

- [[wiki/concepts/gestione-consensi-applicativo\|Gestione Consensi - Applicativo]] §Area Cittadino (CDU-06)
- [[wiki/concepts/informativa\|Informativa Consenso]]
- [[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|Risposte MF SRS v3]] tema D (MF47R46, MF49R48, MF51R50)
- Correlato: [[wiki/docs/adr/ADR-011-merge-cdu-04-05-cittadino\|ADR-011-merge-cdu-04-05-cittadino]] merge CDU-04/05 (UX cittadino semplificata), [[wiki/docs/adr/ADR-015-storicizzazione-immutabile\|ADR-015-storicizzazione-immutabile]] storicizzazione (download non altera storico)
