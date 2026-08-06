---
{"dg-publish":true,"permalink":"/wiki/docs/adr/adr-012-notificatore-deleghe-post-completato/","title":"Notifica cittadino post-acquisizione via Notificatore di Deleghe (NON UNP), dopo COMPLETATO","tags":["notifica","notificatore-deleghe","unp","mf33","batch-01","timing"],"dg-note-properties":{"adr":12,"title":"Notifica cittadino post-acquisizione via Notificatore di Deleghe (NON UNP), dopo COMPLETATO","status":"accepted","date":"2026-05-14","deciders":["Marco Forneris","Exprivia"],"supersedes":[],"superseded-by":[],"tags":["notifica","notificatore-deleghe","unp","mf33","batch-01","timing"],"related_wiki":["[[Sistemi Esterni Integrati]]","[[wiki/concepts/batch-processes\|Processi Batch — BATCH-01, BATCH-02, BATCH-03]]","[[Gestione Consensi - Applicativo]]"],"sources":["[[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|Risposte MF SRS v3]] MF33R31"]}}
---


# ADR-012: Notifica cittadino via Notificatore di Deleghe post-COMPLETATO

## Status

`accepted` — decisione MF33R31 sulla revisione SRS v3.

## Context

A valle di un atto di consenso del cittadino (CDU-03 rilascio o CDU-04/05 modifica/cambio valore), il sistema deve notificare al cittadino/delegato l'avvenuta acquisizione.

CSI Piemonte dispone di due servizi di notifica distinti:

| Servizio | Scopo principale |
|---|---|
| **Notificatore UNP** (User Notification Platform) | Notifiche applicative generiche multi-canale (email, push, AppIO). Documentato su `gitlab.csi.it/user-notification-platform/unpdocumentazione` |
| **Notificatore di Deleghe** | Servizio dedicato alle conferme nell'ambito atti formali (rilascio/modifica consensi e deleghe) |

Confondere i due porta a:
- Onboarding del servizio sbagliato (credenziali, spec, ticket di accesso)
- Documentazione SRS ambigua sul canale (sezioni §3 e §7 ne risentono)
- Implementazione client REST verso endpoint non corretti

Inoltre, il **timing** della notifica al cittadino è critico: non deve partire prima che le notifiche alle ASR (BATCH-01) siano confermate `COMPLETATO`, altrimenti il cittadino potrebbe ricevere conferma di un atto che il SIA della propria ASR non ha ancora ricevuto.

## Decision

Notifica cittadino post-acquisizione:
- **Canale: Notificatore di Deleghe** (NON Notificatore UNP)
- **Timing: parte SOLO dopo conferma notifica alle aziende** — quando tutti i record `cons_t_notifica` per la variazione raggiungono `INVIATO`/`COMPLETATO` lato BATCH-01

Documentazione obbligatoria:
- Distinguere chiaramente Notificatore di Deleghe vs UNP in SRS §3 e §7 (sezione notifiche)
- Wiki [[wiki/concepts/sistemi-esterni-integrati\|Sistemi Esterni Integrati]] entries separate per i due servizi

## Consequences

### Positive
- Cittadino riceve conferma solo quando l'atto è effettivamente propagato (no race condition percepita)
- Allineamento con altri servizi sanità che usano Notificatore di Deleghe per atti formali
- Notificatore UNP riservato a notifiche generiche (es. promemoria, comunicazioni)

### Negative
- **Dipendenza temporale stretta** fra BATCH-01 e notifica cittadino — se BATCH-01 ritarda/fallisce, anche la notifica cittadino è ritardata
- Necessario monitoraggio orchestrazione (quando tutti i record `cons_t_notifica` per una variazione sono COMPLETATO → trigger notifica)
- Approvvigionamento Notificatore di Deleghe da richiedere a CSI (Sprint 0, ❌ Punti Aperti INT)

### Neutral
- Notificatore UNP resta disponibile per altre notifiche applicative

## Alternatives considered

| Alternativa | Motivo scarto |
|---|---|
| Notificatore UNP (canale unico generico) | Errato per natura dell'atto: rilascio consenso è atto formale, non notifica generica |
| Notifica sincrona al cittadino al click "Salva" | Risk race condition: cittadino confermato ma BATCH-01 in errore → divergenza percepita |
| Notifica indipendente (fire-and-forget al click) | Stesso problema |

## Open issues

- ~~INT-05: distinzione formalizzata in SRS — Notificatore di Deleghe ≠ Notificatore UNP~~ ✅ recepito
- ~~Approvvigionamento spec/API Notificatore di Deleghe da CSI~~ ✅ **chiuso (call CSI 06/08/2026):** componente AS-IS legacy, riciclato — nessun nuovo sviluppo richiesto

## References

- [[wiki/concepts/sistemi-esterni-integrati\|Sistemi Esterni Integrati]] §Notificatore di Deleghe + §Notificatore UNP
- [[wiki/concepts/batch-processes\|Processi Batch — BATCH-01, BATCH-02, BATCH-03]] §Notifica al cittadino post-COMPLETATO
- [[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|Risposte MF SRS v3]] MF33R31
- Correlato: [[wiki/docs/adr/ADR-007-batch-01-5min-skip-locked\|ADR-007-batch-01-5min-skip-locked]] BATCH-01 5min (precondizione `COMPLETATO`)
