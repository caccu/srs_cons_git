---
{"dg-publish":true,"permalink":"/wiki/docs/adr/adr-010-cdu-01-split/","title":"Split CDU-01 in CDU-01a Operatore e CDU-01b Cittadino","tags":["cdu-01","autenticazione","profili","mf16","mf18","gasp-salute","pua"],"dg-note-properties":{"adr":10,"title":"Split CDU-01 in CDU-01a Operatore e CDU-01b Cittadino","status":"accepted","date":"2026-05-14","deciders":["Marco Forneris","Exprivia"],"supersedes":[],"superseded-by":[],"tags":["cdu-01","autenticazione","profili","mf16","mf18","gasp-salute","pua"],"related_wiki":["[[Gestione Consensi - Applicativo]]","[[GASP Salute]]","[[Sistemi Esterni Integrati]]"],"sources":["[[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|Risposte MF SRS v3]] MF16R15, MF18R17"]}}
---


# ADR-010: Split CDU-01 — CDU-01a Operatore + CDU-01b Cittadino

## Status

`accepted` — decisione MF16R15 + MF18R17 sulla revisione SRS v3.

> 🔄 **Nota di scope (call CSI 06/08/2026, [[wiki/docs/adr/ADR-021-perimetro-solo-operatore\|ADR-021]]):** la distinzione tecnica CDU-01a/01b resta corretta, ma **solo CDU-01a (Operatore) è in scope di sviluppo** di questo progetto. CDU-01b (Cittadino) descrive un accesso che esiste ma non è un deliverable di questo progetto.
>
> 🔄 **Correzione profilo (call CSI 06/08/2026):** **un solo profilo Operatore**, non selezione multipla — corregge il punto "Selezione profilo applicativo Configuratore visibile (es. Operatore Sanitario, Operatore Back Office)" in Decision sotto. Nessun nuovo profilo da creare in PUA/Configuratore.

## Context

CDU-01 originale (bozza v2) era un caso d'uso unico "Autenticazione" con scenario misto per cittadino e operatore. Limiti:

- IdP, profilo applicativo, precondizioni e flow sono diversi fra cittadino (SPID/CIE via [[wiki/concepts/gasp-salute\|GASP Salute]]) e operatore (PUA via RUPAR/IRIDE)
- La selezione del profilo applicativo Configuratore esiste **solo lato operatore** — cittadino NON è profilo Configuratore (MF7R5)
- Webapp distinte richiedono CDU distinti per leggibilità del SRS e dei test
- Precondizioni miscelate in un solo CDU rendono ambigue le validazioni

## Decision

CDU-01 viene diviso in due sotto-scenari espliciti:

**CDU-01a — Accesso Operatore**
- Accesso: RUPAR / IRIDE / SPID via PUA
- ~~Selezione profilo applicativo Configuratore visibile (es. Operatore Sanitario, Operatore Back Office)~~ ⚠️ **corretto 06/08/2026:** un solo profilo Operatore, nessuna selezione multipla
- Precondizioni: operatore censito in Configuratore Regionale
- CDU coinvolti a valle: CDU-05, CDU-07÷CDU-14

**CDU-01b — Accesso Cittadino** ⚠️ *fuori scope di sviluppo — vedi nota sopra*
- Accesso: SPID / CIE su **webapp separata** (dedicata cittadino)
- Nessuna profilazione iniziale Configuratore (cittadino non è profilo applicativo Configuratore — MF7R5)
- Precondizioni: cittadino con identità digitale attiva
- Pulsante "Deleghe" per operare come delegato (MF20R19) — scenario interno, non profilazione Configuratore
- CDU coinvolti a valle: CDU-02÷CDU-06

Precondizioni separate per scenario.

## Consequences

### Positive
- Specifica SRS leggibile: una sezione per scenario, niente miscelazione
- Test plan separato e tracciabile: scenari operatore e cittadino indipendenti
- Componenti di autenticazione configurati distintamente (Spring Security: 2 SecurityFilterChain)
- Webapp dedicata cittadino con flow SPID/CIE pulito

### Negative
- Due CDU da implementare, documentare e mantenere
- Configurazione Spring Security più articolata (2 filter chain)

### Neutral
- Coerente con esistenza di 2 webapp distinte (cittadino + operatore)

## Alternatives considered

| Alternativa | Motivo scarto |
|---|---|
| CDU-01 unico con varianti | SRS confuso; precondizioni non separabili; ambiguità su profilazione |
| CDU-01a/b/c per delegato | Deleghe è scenario interno a CDU-01b, non CDU autonomo (MF22R21) |

## Open issues

Dipende da [[wiki/concepts/gasp-salute\|GASP Salute]] (rischio critico aperto):
- ID-01: protocollo OIDC vs SAML2 → CDU-01b bloccato finché non definito (vedi [[wiki/analyses/analysis-2026-05-14-punti-aperti-csi\|Punti Aperti CSI]] §1)

## References

- [[wiki/concepts/gestione-consensi-applicativo\|Gestione Consensi - Applicativo]] §CDU-01 split
- [[wiki/concepts/gasp-salute\|GASP Salute]]
- [[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|Risposte MF SRS v3]] MF16R15, MF18R17, MF7R5
- Correlato: [[wiki/docs/adr/ADR-009-eliminazione-sistemats\|ADR-009-eliminazione-sistemats]] no SistemaTS (CDU-07 a valle CDU-01a), [[wiki/docs/adr/ADR-011-merge-cdu-04-05-cittadino\|ADR-011-merge-cdu-04-05-cittadino]] merge CDU-04/05 (coerenza UX cittadino)
