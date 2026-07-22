---
{"dg-publish":true,"permalink":"/wiki/analyses/analysis-2026-05-06-openapi-cdu-15-16/","title":"OpenAPI CDU-15/16 — Descrizione e Stato Consolidamento","tags":["openapi","cdu-15","cdu-16","sia","rest","api","draft","consolidamento","sicurezza"],"dg-note-properties":{"title":"OpenAPI CDU-15/16 — Descrizione e Stato Consolidamento","aliases":["OpenAPI CDU-15/16 — Descrizione e Stato Consolidamento"],"type":"analysis","tags":["openapi","cdu-15","cdu-16","sia","rest","api","draft","consolidamento","sicurezza"],"created":"2026-05-06","updated":"2026-05-14","sources":["2026-03-02-conspref-srs-v1-revised","2026-03-02-domande-srs-csi-v02","2019-06-01-webservice-consenso-regionale-v03"],"related":["[[analysis-2026-05-06-checklist-avvio-progetto|Checklist Avvio Progetto — Gestione Consensi]]","[[valutazione-qualita-srs-consensi|Valutazione Qualità SRS — Gestione Consensi]]","[[wiki/analyses/analysis-gap-as-is-to-be\|Analisi Gap AS-IS → TO-BE — Gestione Consensi]]","[[Sistemi Esterni Integrati]]","[[batch-processes|Processi Batch — BATCH-01, BATCH-02, BATCH-03]]","[[ciclo-vita-consenso|Ciclo di Vita del Consenso]]","[[wiki/concepts/sicurezza-cdu-15-16\|Sicurezza CDU-15-16 — Modello Autorizzazione per Ente]]"]}}
---


# OpenAPI CDU-15/16 — Descrizione e Stato Consolidamento

**File spec:** `wiki/analyses/openapi-cdu-15-16-v0.1.yaml`
**Versione corrente:** `0.1-DRAFT`
**Stato:** Bozza interna — non ancora condivisa con ASR
**Fonte SRS:** CONSPREF-SRS-V1.0 bozza v2, §6.15–6.16

---

## Cos'è questo documento

Specifica OpenAPI 3.0 delle due API REST esposte dal sistema Gestione Consensi verso i **Sistemi Informativi Aziendali (SIA)** delle ASR del Piemonte.

Queste API sono il canale **inbound** (SIA → Regionale): i SIA le chiamano per leggere configurazioni e stato consensi. Non vanno confuse con il canale **outbound** (BATCH-01 → SIA via SOAP), che usa il WSDL AS-IS invariato ([[wiki/sources/2019-06-01-webservice-consenso-regionale-v03\|Specifica WebService ConsensoRegionaleAziendale v03 (AS-IS)]]).

---

## Endpoint esposti

### CDU-15 — Stato consenso

```
GET /api/v1/consensi/stato
    ?codice_fiscale={CF}
    &codice_consenso={cod}
    &codice_ente={ente}
```

**Scopo:** Un SIA verifica lo stato corrente di un consenso per un dato cittadino.

**Fonte dati:**
- `cons_t_consenso` — ultimo record valido
- `cons_d_informativa` — informativa associata

**Response 200 (campi):**

| Campo | Tipo | Note |
|---|---|---|
| `codice_fiscale` | string | CF assistito |
| `codice_consenso` | string | Tipo consenso (es. ROL) |
| `codice_ente` | string | Codice ASR richiedente |
| `stato_consenso` | enum | Vedi stati sotto |
| `valore_consenso` | enum SI/NO/null | null se NON_ESPRESSO |
| `data_espressione` | datetime UTC | Quando il cittadino ha scelto |
| `data_inizio_validita_consenso` | date/null | Inizio validità |
| `data_fine_validita_consenso` | date/null | Fine validità (null = attivo) |
| `informativa.id` | long | ID informativa corrente |
| `informativa.versione` | string | Versione informativa |
| `informativa.data_decorrenza` | date | Decorrenza informativa |
| `informativa.data_scadenza` | date/null | Scadenza informativa (null = attiva) |

**HTTP status:** 200, 400, 401, 403, 404, 500

---

### CDU-16 — Configurazione ente

```
GET /api/v1/configurazione/{codice_ente}
```

**Scopo:** Un SIA scarica la configurazione completa dei consensi per il proprio ente — tipi attivi, informative correnti, endpoint SOAP di notifica.

**Fonte dati:**
- `cons_d_sotto_tipo_cons` — tipi consenso
- `cons_d_informativa` — informative attive
- `cons_r_asr_endpoint` + `cons_t_endpoint` — endpoint notifica
- `cons_r_sotto_tipo_cons_asr_endpoint` — join tipo-ente-endpoint

**Response 200 (struttura):**

```
consensi_attivi[]
  ├── codice_consenso
  ├── descrizione
  ├── tipo (AZIENDALE | REGIONALE | NAZIONALE)
  ├── informativa
  │   ├── id
  │   └── data_decorrenza
  └── endpoints_notifica[]
      ├── endp_id
      └── endp_url  ← stessi URL che usa BATCH-01 per notifiche SOAP
```

**HTTP status:** 200, 401, 403, 404, 500

---

## Sicurezza

**Schema:** Bearer JWT — OAuth2 Client Credentials flow. **Niente API Gateway/Manager CSI** (decisione §3.2 SRS v3): security gestita interamente a livello applicativo (Spring Security).

Il SIA chiamante ottiene un token JWT dall'Authorization Server CSI Piemonte, incluso nell'header:
```
Authorization: Bearer <token>
```

Il token è emesso/validato dall'API Manager APIMBBONE. ✅ **Aggiornamento call 20/07/2026:** l'autorizzazione per ente si poggia sul **`codice_ente` inoltrato dal Gateway APIM** (non su una tabella applicativa): `EnteAuthorizationFilter` + `WHERE codice_ente` nel repository. La tabella `cons_t_client_ente` è **fuori scope V1.0** (estensione futura per multi-ente).

> Modello completo, claim minimi, pseudocodice filter, schema tabella, matrice attacchi/mitigazioni, audit log, testo proposto per SRS: **[[wiki/concepts/sicurezza-cdu-15-16\|Sicurezza CDU-15-16 — Modello Autorizzazione per Ente]]** (risposta tecnica al commento cliente TR30).

---

## Gestione errori

Tutti gli endpoint usano **RFC 7807** (Problem Details for HTTP APIs):

```json
{
  "type": "https://gestioneconsensi.csi.piemonte.it/errors/not-found",
  "title": "Not Found",
  "status": 404,
  "detail": "Nessun consenso trovato per i parametri specificati.",
  "instance": "/api/v1/consensi/stato"
}
```

Content-type: `application/problem+json`

---

## Nota semantica critica per integratori SIA

> ⚠️ **Stato SCADUTO — comportamento cambiato tra AS-IS e TO-BE**

| | AS-IS | TO-BE |
|---|---|---|
| Chi imposta SCADUTO | Logica acquisizione durante SRV-01 | BATCH-02 su scadenza informativa |
| Quando | Durante acquisizione (sincrono) | A scadenza informativa (asincrono, notturno) |
| Notifica ASR | Sincrona durante acquisizione | Nessuna notifica per SCADUTO — solo per ANNULLATO |

I SIA che hanno logica basata sul vecchio comportamento devono aggiornare i propri processi interni. Vedi [[wiki/analyses/analysis-gap-as-is-to-be\|Analisi Gap AS-IS → TO-BE — Gestione Consensi]] §Cambiamenti semantici critici.

---

## Enum StatoConsenso

| Valore | Significato | Notifica BATCH-01 inviata? |
|---|---|---|
| `NON_ESPRESSO` | Cittadino non ha mai scelto | No |
| `ATTIVO` | Consenso valido, informativa corrente | Sì (su variazione) |
| `NEGATO` | Rifiuto esplicito, informativa corrente | Sì (su variazione) |
| `SCADUTO` | Informativa cambiata, `annulla_consensi=NO` | **No** |
| `ANNULLATO` | Informativa cambiata, `annulla_consensi=SI` | **Sì** |

---

## Integrazione con Spring Boot (note sviluppatori)

L'SRS raccomanda generazione automatica degli stub dal YAML:

```xml
<!-- pom.xml -->
<plugin>
  <groupId>org.openapitools</groupId>
  <artifactId>openapi-generator-maven-plugin</artifactId>
  <configuration>
    <inputSpec>${project.basedir}/src/main/resources/openapi-cdu-15-16.yaml</inputSpec>
    <generatorName>spring</generatorName>
    <configOptions>
      <useSpringBoot3>true</useSpringBoot3>
      <interfaceOnly>true</interfaceOnly>
    </configOptions>
  </configuration>
</plugin>
```

Swagger UI esposta in ambienti non produttivi su `/swagger-ui.html` tramite **SpringDoc OpenAPI**:

```xml
<dependency>
  <groupId>org.springdoc</groupId>
  <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
</dependency>
```

---

## Stato consolidamento

### ✅ Consolidato — ricavato da SRS

| # | Elemento | Fonte |
|---|---|---|
| C01 | Endpoint CDU-15: `GET /api/v1/consensi/stato` | SRS §6.15 |
| C02 | Endpoint CDU-16: `GET /api/v1/configurazione/{codice_ente}` | SRS §6.16 |
| C03 | Query params CDU-15: `codice_fiscale`, `codice_consenso`, `codice_ente` | SRS §6.15 scenario principale |
| C04 | Path param CDU-16: `codice_ente` | SRS §6.16 scenario principale |
| C05 | Tutti i campi response CDU-15 (9 campi + nested informativa) | SRS §6.15 response 200 |
| C06 | Struttura response CDU-16 (consensi_attivi + informativa + endpoints_notifica) | SRS §6.16 response 200 |
| C07 | Auth: Bearer JWT OAuth2 Client Credentials | SRS §6.15–6.16 + Q&A CSI #6 |
| C08 | Errori RFC 7807 su tutti gli endpoint | SRS §4.x |
| C09 | HTTP status codes CDU-15: 200/400/401/403/404/429/500 | SRS §6.15 |
| C10 | HTTP status codes CDU-16: 200/401/403/404/429/500 | SRS §6.16 |
| C11 | Enum StatoConsenso: 5 valori (NON_ESPRESSO/ATTIVO/NEGATO/SCADUTO/ANNULLATO) | SRS §5 macchina a stati |
| C12 | Enum ValoreConsenso: SI/NO/null | SRS §5 |
| C13 | Enum tipo consenso: AZIENDALE/REGIONALE/NAZIONALE | SRS §5 + CONSPREF-SRS-01-V03 |
| C14 | Base path `/api/v1/` | SRS §6.15 (esplicito) |
| C15 | Pattern CF: regex `^[A-Z]{6}[0-9]{2}[A-Z][0-9]{2}[A-Z][0-9]{3}[A-Z]$` | standard italiano |
| C16 | Tabelle DB sorgente per ogni endpoint | SRS §6.15–6.16 + §8 |
| C17 | `endp_url` in CDU-16 = stessi URL usati da BATCH-01 | SRS §7.1 + §6.16 |
| C18 | Generazione stub via `openapi-generator-maven-plugin` | SRS §4 prerequisiti OpenAPI |
| C19 | Swagger UI via SpringDoc su `/swagger-ui.html` (non-prod) | SRS §4 |

---

### ❌ Da consolidare con CSI Piemonte

| ID | Informazione mancante | Impatto sul YAML | Priorità |
|---|---|---|---|
| TODO-M1 | URL Authorization Server CSI + JWKS endpoint | `securitySchemes.bearerAuth` — ora placeholder | 🔴 Pre go-live |
| TODO-M2 | Scope OAuth2 richiesto per client SIA (es. `consensi:read`) | Documentazione integrazione SIA | 🔴 Pre go-live |
| TODO-M3 | Paginazione su CDU-16 richiesta? | Aggiunta `page`/`size` query params + `PaginatedResponse` schema | 🟠 Sprint 2 |
| TODO-M4 | SLA: rate limit per client, timeout massimo risposta | Header `X-RateLimit-*`, note NFR | 🟡 Pre UAT |
| TODO-M5 | Lista ASR coinvolte + `client_id` per env DEV/PROD | Env-specific server URLs, test setup | 🟡 Sprint 3 |

---

### 🔧 Da produrre internamente (Exprivia)

| ID | Attività | Scadenza raccomandata |
|---|---|---|
| P01 | Validazione YAML con Swagger Editor / `swagger-codegen validate` | Immediato |
| P02 | Aggiornare versione da `0.1-DRAFT` a `0.1` dopo validazione CSI bozza | Post prima review CSI |
| P03 | Copia YAML in `src/main/resources/` del progetto Spring Boot | Sprint 1 |
| P04 | Configurare `openapi-generator-maven-plugin` per generazione stub | Sprint 1 |
| P05 | Implementare endpoint CDU-15 e CDU-16 (Sprint 6 nel piano) | Sprint 6 |
| P06 | Condividere bozza con referenti tecnici ASR per feedback | **Anticipare a Sprint 2** (non Sprint 6) |
| P07 | Compilare TODO-M1÷M5 e rimuovere `-DRAFT` dalla versione | Prima di Sprint 6 |
| P08 | Test integrazione con SIA DEV (mock → reale) | Sprint 8 |

---

## Cronologia versioni

| Versione | Data | Autore | Modifiche |
|---|---|---|---|
| 0.1-DRAFT | 2026-05-06 | Exprivia (wiki LLM) | Prima bozza da SRS §6.15–6.16 |

---

## Prossimi passi

1. **Subito:** validare il YAML con Swagger Editor (incolla `openapi-cdu-15-16-v0.1.yaml`)
2. **Sprint 0:** richiedere a CSI URL Auth Server (TODO-M1) e scope (TODO-M2)
3. **Sprint 1:** copiare YAML nel progetto, configurare plugin Maven
4. **Sprint 2:** condividere bozza con ASR + raccogliere feedback su TODO-M3/M4/M5
5. **Prima di Sprint 6:** compilare tutti i TBD, rilasciare versione `1.0`

---

## ADR correlati

| ADR | Decisione |
|---|---|
| [ADR-004](ADR-004-no-api-gateway.md) | No API Gateway — sicurezza applicativa |
| [ADR-005](ADR-005-sicurezza-cdu-15-16.md) | Modello sicurezza CDU-15/16 OAuth2 CC + JWT |
| [ADR-018](ADR-018-rfc-7807-error-response.md) | RFC 7807 error response |
| [ADR-016](ADR-016-scaduto-async-batch-02.md) | Stato SCADUTO async (semantica cambiata vs AS-IS) |
