---
{"dg-publish":true,"permalink":"/wiki/concepts/sicurezza-cdu-15-16/","title":"Sicurezza CDU-15-16 — Modello Autorizzazione per Ente","tags":["sicurezza","cdu-15","cdu-16","oauth2","jwt","multi-tenancy","spring-security","tr30"],"dg-note-properties":{"title":"Sicurezza CDU-15-16 — Modello Autorizzazione per Ente","aliases":["Sicurezza CDU-15-16 — Modello Autorizzazione per Ente"],"type":"concept","tags":["sicurezza","cdu-15","cdu-16","oauth2","jwt","multi-tenancy","spring-security","tr30"],"created":"2026-05-14","updated":"2026-07-13","sources":["2026-03-02-conspref-srs-v1-revised","2026-03-02-domande-srs-csi-v02"],"related":["[[wiki/analyses/analysis-2026-05-06-openapi-cdu-15-16\|analysis-2026-05-06-openapi-cdu-15-16]]","[[Sistemi Esterni Integrati]]","[[Architettura IaaS]]","[[CSI Piemonte]]","[[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|analysis-2026-05-14-risposte-mf-srs-v3]]","[[Gestione Consensi - Applicativo]]"]}}
---


# Sicurezza CDU-15/16 — Modello Autorizzazione per Ente

**Origine:** risposta tecnica al commento cliente **TR30** sulla revisione SRS bozza v3 (PDF righe 4456–4459, sez. §6.16). Rinumerato come **TR58** nella revisione `CONSPREF-SRS-V1.0-revised_bozza_v3_CSI_lavorazione.pdf` (MF59-62R58). Vedi mapping in [[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|analysis-2026-05-14-risposte-mf-srs-v3]].

**Quesito originale TR30 / TR58:**
> "Si richiede di dettagliare meglio il comportamento del WS e la security (come si lega la possibilità di vedere unicamente i propri ws, passiamo dal API manger?)"

---

## 1. Modello di sicurezza AS-IS / TO-BE — API Manager

> ⚠️ **Aggiornamento verbale 11/06/2026:** Il modello è stato chiarito in riunione. La decisione "No API Manager" da Q&A CSI #6 vale per i **fruitori AS-IS esistenti**. Il TO-BE introduce **doppia esposizione** con API Manager per nuovi fruitori esterni.

### 1.1 AS-IS — Certificati firmati (invariato)

Le richieste provenienti dalle ASR esistenti sono firmate tramite **certificato** (non token), risultando **non ripudiabili**. Questo schema rimane invariato per i fruitori AS-IS.

### 1.2 TO-BE — Doppia esposizione

**Per nuovi fruitori esterni:** API esposte tramite **API Manager CSI Piemonte (APIMBBONE)**; sicurezza demandata all'API Manager con **token OAuth2 (`client_credentials`)** rilasciato dal Key Manager (cfr. §1.4 — non una firma JWS lato nostro).

| Canale | Auth | Fruitori |
|---|---|---|
| Certificato (AS-IS) | Firma certificato X.509 — non ripudiabile | SIA ASR esistenti |
| API Manager (TO-BE) | Token OAuth2 (`client_credentials`) via API Manager APIMBBONE | Nuovi fruitori esterni |

CSI Piemonte fornirà componente bridge tra API Manager e prodotto. Forneris abilitato come guest su "deleghe API" per consultare l'esempio.

**Pattern confermato dall'esempio DelegheApi (immagine verbale 11/06/2026):**

```
Consumer (es. Telemedicina / Gestione Consensi)
    → getDelegantiService
    → API-Piemonte  (accreditamento tramite portale)
    → DelegheApi
```

Gestione Consensi seguirà lo stesso schema: accreditamento sul portale API-Piemonte, chiamate instradate via API Manager. Vedi [[wiki/concepts/sistemi-esterni-integrati\|Sistemi Esterni Integrati]] §Gestione Deleghe.

> ✅ **Aggiornamento 07/2026 (doc `LG_APIMBBONE-APIStore_Sottoscrizione`):** il token è **OAuth2** (grant `client_credentials`) **rilasciato e gestito dal Key Manager dell'API Manager CSI (APIMBBONE)**; i servizi SIA (CDU-15/16/17) sono esposti **dietro l'API Gateway** dell'APIM. Non è quindi un Authorization Server a sé né una firma JWS da implementare lato nostro. Dettaglio nel §1.4.

---

### 1.4 API Manager CSI (APIMBBONE) — modello confermato (07/2026)

Fonte: `LG_APIMBBONE-APIStore_Sottoscrizione_EXT_SI_V1.0.0` (guida per System Integrator esterni).

Il token per i servizi REST verso i SIA è **rilasciato e gestito dall'API Manager CSI (APIMBBONE)**, non da un Authorization Server a sé né dal nostro backend. I servizi Gestione Consensi (CDU-15/16/17) sono **esposti dietro l'API Gateway** dell'APIM.

**Componenti APIMBBONE:**
- **API Store** — portale di accreditamento e sottoscrizione per i fruitori
- **Publisher** — dove il provider pubblica l'API (endpoint backend, throttling)
- **API Gateway** — proxy runtime: tutte le chiamate passano di qui; applica rate limiting/throttling e instrada al backend
- **Key Manager** — rilascia/valida i token OAuth2 (verifica `client_id`/`client_secret`)
- **Traffic Manager** — politiche di traffico in real time

**OAuth2 — grant supportati:** `client_credentials` (usato dai SIA), `authorization_code`, `refresh_token`, `resource_owner_password`.

**Flusso di onboarding del fruitore SIA:**
1. Accreditamento allo **API Store** (SPID per aziende IT; CSI/fornitori con VPN via IdP "Unified Communication").
2. Creazione **applicazione** (contenitore delle sottoscrizioni) → validazione admin → chiavi OAuth (`client_id`/`client_secret`).
3. **Sottoscrizione** all'API — **richiede lo swagger (OpenAPI) del servizio**.
4. Generazione **token** (`client_credentials`) alla token API.
5. Invocazione dell'API **via gateway** con `Authorization: Bearer <token>`.

**Store / endpoint (pattern):**
- Store **test** `https://tst-api-<ente>-store.csi.it/` — **accessibile solo da rete interna CSI/VPN**; per i fruitori esterni le operazioni sono **mediate dal referente CSI**, che fornisce le chiavi OAuth. Gli **endpoint del gateway di test** (token + API) sono raggiungibili **da internet solo via https**.
- Store **prod** `https://api-<ente>-store.csi.it/`.

> **Implicazioni per il progetto:**
> - Va **prodotto e consegnato lo swagger (OpenAPI) di CDU-15/16/17** per poter sottoscrivere l'API sull'APIM (prerequisito).
> - Token e rate limiting/throttling sono **gestiti dall'APIM** → il backend può **non** validare il JWT via JWKS né implementare `bucket4j`; resta a noi l'**isolamento dei dati per ente**. Da definire con CSI **quali header/claim il gateway inoltra** al backend per mappare consumer → `codice_ente`.
> - Conferma la **doppia esposizione** (§1.2): per i SIA/nuovi fruitori si passa dall'API Manager; per l'app interna Frontend→Backend resta l'integrazione diretta (ADR-004).

---

### 1.3 Riferimento storico — Decisione SRS v3 §3.2 (valida AS-IS)

Decisione confermata da CSI ([[wiki/sources/2026-03-02-domande-srs-csi-v02\|Domande SRS Consensi — Revisione CSI V02]] §Q&A #6) e formalizzata in SRS v3 §3.2:

> "il progetto non adotterà l'API Gateway centralizzato del CSI Piemonte come punto d'ingresso esterno. L'architettura adotta un modello di integrazione diretta: le chiamate HTTP [...] vengono instradate direttamente ai Servizi Backend Spring Boot 3, senza intermediari gateway esterni al progetto. La sicurezza delle API (autenticazione e autorizzazione) è interamente gestita a livello applicativo tramite Spring Security"

*(Applicabile ai soli fruitori AS-IS — superato per TO-BE dal verbale 11/06/2026)*

### Implicazioni operative

> ⚠️ Tabella aggiornata al modello APIMBBONE confermato (07/2026): token e traffico sono gestiti dall'API Manager, non più dal backend (cfr. §1.4).

| Funzione | Chi la gestisce (TO-BE, SIA) | Componente |
|---|---|---|
| Rilascio/validazione token OAuth2 | **API Manager (APIMBBONE)** | Key Manager APIM |
| Rate limiting / throttling | **API Manager (APIMBBONE)** | Traffic Manager APIM |
| Autorizzazione granulare **per ente** | Backend applicativo | Filter custom `EnteAuthorizationFilter` + `cons_t_client_ente` |
| TLS termination | Piattaforma / APIM Gateway | Gateway APIM + layer rete IaaS CSI/Nivola |
| WAF | Piattaforma | Layer di rete CSI/Nivola |
| Audit log applicativo | Backend applicativo | Logger applicativo strutturato JSON |

---

## 2. Flusso end-to-end (SIA ASR → Gestione Consensi)

> Nota (07/2026): token rilasciato/validato dall'**API Manager APIMBBONE** (Key Manager + Gateway). Il backend riceve la richiesta **già autenticata dal gateway** e si concentra sull'isolamento per ente; la validazione JWT via JWKS lato backend potrebbe non essere necessaria (da confermare con CSI quali header/claim il gateway inoltra). Cfr. §1.4.

```
[SIA ASR x]
   │ 1. POST /token  (grant_type=client_credentials, client_id, client_secret)
   ▼
[API Manager APIMBBONE — Key Manager]  ──► access token OAuth2 (client_credentials)
   │
   │ 2. GET /api/v1/...  +  Header: Authorization: Bearer <token>
   ▼
[API Gateway APIMBBONE]  ──► valida token, rate limiting/throttling, instrada al backend
   │
   ▼
[Spring Boot Backend Gestione Consensi]
   ├─ A) Spring Security filter chain
   │     ├─ Validazione firma JWT (chiave pubblica AS via JWKS endpoint)
   │     ├─ Check claim: exp, iss, aud
   │     └─ Estrazione client_id
   ├─ B) EnteAuthorizationFilter (custom)
   │     ├─ Lookup tabella cons_t_client_ente: client_id → codice_ente_autorizzato
   │     ├─ Estrazione codice_ente dalla request (path/query)
   │     └─ Confronto: mismatch → 403 Forbidden
   ├─ C) Business logic (RestController CDU-15/16)
   │     └─ Repository query SEMPRE con WHERE codice_ente = :authorizedEnte
   │        (preso da SecurityContext, NON dal parametro request)
   └─ D) Response 200 / 401 / 403 / 404 / 500
```

---

> ⚠️ **Inquadramento (07/2026) delle sezioni §3–§8.** Le sezioni seguenti descrivono il disegno di sicurezza **pre-APIM**. Con il modello **API Manager APIMBBONE confermato** (§1.4):
> - **Emissione/validazione token** (ex Livello A, JWKS, Authorization Server) e **rate limiting** (ex `bucket4j`) sono **a carico dell'APIM** → i riferimenti a JWKS/`bucket4j`/Authorization Server qui sotto sono **superati** e mantenuti come storico.
> - **Restano validi e a nostro carico** l'**isolamento per ente** (Livello B `cons_t_client_ente` + Livello C `WHERE codice_ente`) e l'**audit log applicativo**.
> - Il backend riceverà dal **Gateway APIM** l'identità del consumer via header/claim (da concordare con CSI) da cui derivare il `codice_ente`.
> - Il **testo per l'SRS** effettivamente recepito in `CONSPREF-SRS-V1.0-revised_v5` §6.16 è la versione **APIM** (vedi §8, riquadro aggiornato).

## 3. Isolamento dati per ente — difesa a 3 livelli

Il requisito "vedere unicamente i propri WS/consensi" non si poggia sul trasporto ma su tre controlli sovrapposti.

### Livello A — Identità del client

- Ogni SIA ASR riceve coppia `client_id` + `client_secret` dall'AS CSI Piemonte (procedura onboarding da concordare).
- AS emette JWT firmato (RS256/ES256) con claim `client_id` immodificabile.
- La chiave pubblica AS è pubblicata via endpoint JWKS; il backend la usa per verificare la firma.
- Nessun client può spacciarsi per altro ente: rotazione/revoca gestita lato AS.

### Livello B — Mapping client_id → codice_ente

- Tabella applicativa `cons_t_client_ente` (da aggiungere in SRS §8 modello dati) lega ogni `client_id` a **uno e un solo** `codice_ente` autorizzato.
- Lookup eseguito dal filter, non encoded nel JWT → consente revoca senza riemissione token, audit storico, ruoli speciali (es. ente "aggregatore" multi-ente in futuro).
- Schema tabella proposto:

| Colonna | Tipo | Note |
|---|---|---|
| `client_id` | varchar(128) PK | Stringa fornita da AS CSI |
| `codice_ente` | varchar(10) NOT NULL | FK a `cons_d_ente` |
| `scopes` | text[] | Es. `{"consensi:read"}` |
| `data_attivazione` | timestamp NOT NULL | |
| `data_revoca` | timestamp NULL | NULL = attivo |
| `note` | text | Riferimento contratto/ticket onboarding |

### Livello C — Filtro query (defense in depth)

- Tutte le query repository di CDU-15/16 prendono `codice_ente` da `SecurityContext`, **mai dal parametro request**.
- Anche se il filter B fosse bypassato per bug, la query `WHERE codice_ente = :authEnte` impedisce data leak.
- Pattern repository:

```java
@Query("SELECT c FROM Consenso c WHERE c.codiceEnte = :authEnte AND c.codiceFiscale = :cf")
Optional<Consenso> findByCfAndAuthEnte(@Param("cf") String cf, @Param("authEnte") String authEnte);
```

---

## 4. Matrice attacchi/mitigazioni

| Attacco | Mitigazione | Livello |
|---|---|---|
| Token forgiato | Firma JWT validata con chiave pubblica AS (JWKS) | A |
| Replay di token scaduto | Check claim `exp` | A |
| Token rubato da altro ente | `client_id` legato a singolo `codice_ente` in DB | B |
| Manipolazione `codice_ente` in URL | Compare URL ente vs ente autorizzato → 403 | B |
| Bug nel filter applicativo | Repository forza `WHERE codice_ente = :authEnte` | C |
| Enumerazione codici fiscali | Rate limit applicativo (`bucket4j`) + audit log | App |
| Credenziale `client_secret` esposta | Rotation + revoca via `data_revoca` in tabella | A+B |
| MITM | TLS obbligatorio su Ingress, HSTS | Piattaforma |

---

## 5. Specifica tecnica per SRS

### 5.1 Security Scheme OpenAPI

```yaml
components:
  securitySchemes:
    bearerAuth:
      type: oauth2
      flows:
        clientCredentials:
          tokenUrl: https://auth.csi.piemonte.it/oauth2/token   # ⚠️ URL da confermare con CSI
          scopes:
            consensi:read: Lettura stato consensi e configurazioni del proprio ente

security:
  - bearerAuth: [consensi:read]
```

### 5.2 Claim JWT minimi

```json
{
  "iss": "https://auth.csi.piemonte.it",
  "aud": "gestione-consensi",
  "sub": "sia-asl-to1-prod",
  "client_id": "sia-asl-to1-prod",
  "scope": "consensi:read",
  "iat": 1736937600,
  "exp": 1736941200
}
```

**TTL raccomandato:** 3600 secondi (da confermare con CSI).

### 5.3 Pseudocodice filter Spring Security

```java
@Component
public class EnteAuthorizationFilter extends OncePerRequestFilter {

    private final ClientEnteMapper mapper;

    @Override
    protected void doFilterInternal(HttpServletRequest req,
                                    HttpServletResponse res,
                                    FilterChain chain) throws ServletException, IOException {

        Jwt jwt = (Jwt) SecurityContextHolder.getContext()
                                             .getAuthentication()
                                             .getPrincipal();
        String clientId = jwt.getClaimAsString("client_id");

        String authorizedEnte = mapper.resolveEnte(clientId);
        if (authorizedEnte == null) {
            problem(res, 403, "client_not_registered",
                    "Client non censito nella tabella cons_t_client_ente");
            return;
        }

        String requestedEnte = extractEnteFromRequest(req);
        if (!authorizedEnte.equals(requestedEnte)) {
            problem(res, 403, "ente_mismatch",
                    "Ente richiesto non corrisponde al client autenticato");
            return;
        }

        req.setAttribute("authorizedEnte", authorizedEnte);
        chain.doFilter(req, res);
    }
}
```

### 5.4 HTTP status codes

| Codice | Causa |
|---|---|
| `200 OK` | Token valido, ente match, dato trovato |
| `401 Unauthorized` | Token assente, scaduto o firma invalida |
| `403 Forbidden` | Token valido ma `codice_ente` richiesto ≠ autorizzato, oppure client non censito |
| `404 Not Found` | Ente autorizzato ma nessun consenso/configurazione per i parametri |
| `429 Too Many Requests` | Rate limit applicativo superato (nuovo, conseguenza assenza APIM) |
| `500 Internal Server Error` | Errore non gestito |

Tutti i 4xx/5xx rispettano **RFC 7807** con `application/problem+json`.

---

## 6. Audit log obbligatorio

Ogni invocazione di CDU-15/CDU-16 registra (struttura JSON):

```json
{
  "ts": "2026-05-14T10:35:22.143Z",
  "client_id": "sia-asl-to1-prod",
  "codice_ente_requested": "010",
  "codice_ente_authorized": "010",
  "endpoint": "/api/v1/consensi/stato",
  "params_hash": "sha256:...",
  "outcome": "200",
  "latency_ms": 47,
  "trace_id": "..."
}
```

Codice fiscale **NON** loggato in chiaro (vedi [[wiki/analyses/valutazione-qualita-srs-consensi\|Valutazione Qualità SRS — Gestione Consensi]] §sicurezza, OWASP). Hash dei parametri per ricerca breach post-mortem.

---

## 7. Gap aperti / azioni richieste

### Aggiornamenti SRS richiesti

| # | Sezione SRS | Modifica |
|---|---|---|
| G1 | §6.15, §6.16 | Aggiungere paragrafo "Modello di sicurezza per ente" (testo proposto in §8 sotto) |
| G2 | §3.3 Componenti software | Aggiungere componente `EnteAuthorizationFilter` (Spring Security custom filter) |
| G3 | §3.6 (sicurezza, se esistente) | Documentare rate limiting applicativo `bucket4j` come sostituto APIM |
| G4 | §8 Modello dati | Aggiungere entità `cons_t_client_ente` (schema in §3.2 sopra) |
| G5 | §10 Test plan | Aggiungere caso E2E cross-tenant: client A tenta lettura ente B → atteso 403 |
| G6 | §11 Ops/Monitoring | Definire dashboard audit log con outcome 401/403/429 anomali |

### Punti da chiarire con CSI

| # | Domanda | Blocco |
|---|---|---|
| Q1 | ~~URL produzione/test Authorization Server CSI~~ → **risolto:** token via API Manager APIMBBONE (Store test `tst-api-<ente>-store.csi.it`, gateway https). Resta: **quali header/claim il Gateway inoltra** al backend | ⚠️ Sprint 0 |
| Q2 | ~~Algoritmo firma JWT + URL JWKS~~ → **non più a nostro carico:** token gestito dall'APIM (Key Manager). Prerequisito nostro: **produrre lo swagger (OpenAPI)** per la sottoscrizione | ⚠️ Sprint 0 |
| Q3 | Procedura onboarding nuovo SIA (accreditamento Store, chi crea l'applicazione/chiavi OAuth, chi popola il mapping `cons_t_client_ente`) | Sprint 1 |
| Q4 | TTL token raccomandato e politica refresh | Sprint 1 |
| Q5 | Scope predefiniti CSI o liberi a definire dal progetto? | Sprint 1 |
| Q6 | Politica revoca credenziali compromesse (blacklist? rotation?) | Sprint 2 |

---

## 8. Testo per SRS §6.15 e §6.16 — versione APIM (recepita in v5)

> ⚠️ **Aggiornato 07/2026.** Il riquadro sotto è la versione **APIMBBONE** effettivamente inserita in `CONSPREF-SRS-V1.0-revised_v5` §6.16. La precedente formulazione (Authorization Server + JWKS + `bucket4j`, "senza API Gateway/Manager") è **superata** ed è conservata solo nel log storico.

> **Modello di sicurezza CDU-15/16**
>
> L'accesso ai servizi CDU-15 e CDU-16 è regolato da autenticazione OAuth2 `client_credentials`, con token **emesso e validato dall'API Manager del CSI Piemonte (APIMBBONE)** (Key Manager) e servizi esposti dietro l'**API Gateway** dell'API Manager; l'autorizzazione applicativa a livelli (isolamento per ente) resta a carico del Backend.
>
> L'isolamento dei dati per ente (visibilità unicamente dei propri consensi/configurazioni) è garantito da:
>
> **(a)** token OAuth2 emesso dall'API Manager CSI (APIMBBONE) e da esso validato, associato al `client_id` univoco del SIA chiamante; il backend riceve dal Gateway l'identità del client (header/claim da concordare con CSI) e non gestisce direttamente la validazione JWKS;
>
> **(b)** tabella applicativa `cons_t_client_ente` che lega ogni `client_id` a un singolo `codice_ente` autorizzato, popolata in fase di onboarding del SIA e gestita dall'amministratore di sistema;
>
> **(c)** filtro Spring Security (`EnteAuthorizationFilter`) che confronta il `codice_ente` presente nella request (path/query) con quello autorizzato per il `client_id` autenticato, rispondendo `403 Forbidden` in caso di discrepanza, e repository che applica sempre `WHERE codice_ente = :authorizedEnte` come difesa in profondità.
>
> Il rate limiting/throttling è fornito dall'**API Manager CSI (APIMBBONE — Traffic Manager)** e non è più implementato a livello applicativo. Resta a carico del Backend un audit log strutturato, obbligatorio per ogni invocazione, che registra `client_id`, `codice_ente_requested`, `codice_ente_authorized`, `outcome`, `latency_ms`, `trace_id`. Il codice fiscale non viene loggato in chiaro.
>
> **Prerequisito:** produzione e consegna dello **swagger (OpenAPI)** di CDU-15/16/17 per la sottoscrizione dell'API sull'APIM.

---

## 9. Riferimenti

- Decisione architetturale "no API Gateway": [[wiki/sources/2026-03-02-domande-srs-csi-v02\|Domande SRS Consensi — Revisione CSI V02]] Q&A #6, [[wiki/sources/2026-03-02-conspref-srs-v1-revised\|CONSPREF-SRS-V1.0 revised bozza v2]] §3.2
- Specifica OpenAPI dei due endpoint: [[wiki/analyses/analysis-2026-05-06-openapi-cdu-15-16\|analysis-2026-05-06-openapi-cdu-15-16]]
- Stack tecnologico Spring Boot 3 + Spring Security: [[wiki/sources/2026-03-12-pile-tecnologiche-csi\|Pile Tecnologiche CSI Piemonte]]
- Vincoli ECaaS / Ingress / TLS: [[wiki/sources/2019-06-01-linea-guida-fornitori-cloud-native\|Linee Guida Cloud Native per Fornitori v1.0.1]], [[wiki/concepts/architettura-iaas\|Architettura IaaS]]
- Inventario sistemi consumer (SIA ASR): [[wiki/concepts/sistemi-esterni-integrati\|Sistemi Esterni Integrati]]

---

## ADR correlati

| ADR | Decisione |
|---|---|
| [ADR-004](ADR-004-no-api-gateway.md) | No API Gateway — sicurezza applicativa Spring Security |
| [ADR-005](ADR-005-sicurezza-cdu-15-16.md) | Modello sicurezza CDU-15/16 a 3 livelli (questa concept è la fonte autoritativa) |
| [ADR-018](ADR-018-rfc-7807-error-response.md) | RFC 7807 error response |
| [ADR-006](ADR-006-batch-03-pull-cdu-17.md) | CDU-17 PULL riusa stesso pattern sicurezza |
