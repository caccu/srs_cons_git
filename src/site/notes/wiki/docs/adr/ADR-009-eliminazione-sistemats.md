---
{"dg-publish":true,"permalink":"/wiki/docs/adr/adr-009-eliminazione-sistemats/","title":"Eliminazione SistemaTS dall'integrazione (no fallback ricerca assistito)","tags":["aura","sistemats","integrazione","cdu-07","mf53","mf55"],"dg-note-properties":{"adr":9,"title":"Eliminazione SistemaTS dall'integrazione (no fallback ricerca assistito)","status":"accepted","date":"2026-05-14","deciders":["Marco Forneris","Exprivia"],"supersedes":[],"superseded-by":[],"tags":["aura","sistemats","integrazione","cdu-07","mf53","mf55"],"related_wiki":["[[Sistemi Esterni Integrati]]","[[Gestione Consensi - Applicativo]]","[[wiki/analyses/analysis-gap-as-is-to-be\|Analisi Gap AS-IS → TO-BE]]"],"sources":["[[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|Risposte MF SRS v3]] MF53R52, MF55R54"]}}
---


# ADR-009: Eliminazione SistemaTS dall'integrazione

## Status

`accepted` — decisione MF53R52 + MF55R54 sulla revisione SRS v3.

## Context

La bozza SRS v3 prevedeva, per CDU-07 (ricerca assistito da Operatore), una catena:
1. **AURA** invocato per primo (FindProfiliAnagrafici + getProfiloSanitario)
2. **SistemaTS** come fallback se AURA non trova il CF

Considerazioni emerse durante revisione cliente:
- AURA è l'Anagrafe Unica Regionale Assistiti — l'unica fonte autoritativa per la popolazione assistiti Piemonte
- Se AURA non trova un CF, significa che l'assistito non è registrato come tale in regione — chiamare SistemaTS aggiunge una dipendenza esterna senza valore informativo per il dominio Gestione Consensi
- SistemaTS introduce un secondo protocollo, un secondo set di credenziali, ulteriore latenza e un rischio operativo aggiuntivo

## Decision

**SistemaTS NON è integrato** dal sistema Gestione Consensi TO-BE.

CDU-07 comportamento aggiornato:
- Invoca **solo** AURA (`FindProfiliAnagrafici` + `getProfiloSanitario`)
- Se CF non trovato → messaggio fisso «La ricerca con il filtro fornito non ha prodotto risultati»
- Nessun fallback, nessuna chiamata a SistemaTS

Conseguenza documentale (MF55R54):
- **Eliminare tutti i riferimenti a SistemaTS** in SRS §6.7
- Aggiornare diagrammi di contesto, mappe di integrazione, modello dati
- Wiki: SistemaTS rimosso dal corpus dei sistemi integrati

## Consequences

### Positive
- Una integrazione in meno da provisioning, monitorare, mantenere
- Latenza CDU-07 prevedibile (un solo upstream)
- Niente credenziali SistemaTS da gestire
- Architettura più semplice e auditabile

### Negative
- Se AURA è giù → CDU-07 fallisce completamente, nessun fallback
- Eventuali CF presenti in SistemaTS ma non in AURA non sono raggiungibili (caso ritenuto marginale: AURA è autoritativa regionale)

### Neutral
- Diagrammi e mappe da pulire dal riferimento residuo

## Alternatives considered

| Alternativa | Motivo scarto |
|---|---|
| Mantenere SistemaTS come fallback | Cliente ha esplicitamente eliminato il fallback (MF55R54) |
| SistemaTS solo per CF specifici (es. fuori regione) | Aggiunge complessità senza chiaro caso d'uso ricorrente per Gestione Consensi |

## References

- [[wiki/concepts/sistemi-esterni-integrati\|Sistemi Esterni Integrati]] §AURA
- [[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|Risposte MF SRS v3]] tema E (MF53R52, MF55R54)
- [[wiki/analyses/analysis-gap-as-is-to-be\|Analisi Gap AS-IS → TO-BE]] (stack integrazione semplificato)
- Correlato: [[wiki/docs/adr/ADR-010-cdu-01-split\|ADR-010-cdu-01-split]] split CDU-01 (operatore lato CDU-07/08 coinvolge AURA)
