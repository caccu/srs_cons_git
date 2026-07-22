---
{"dg-publish":true,"permalink":"/wiki/concepts/sicurezza-cdu-15-16/","title":"Sicurezza CDU-15-16 — Modello Autorizzazione per Ente","tags":["sicurezza","cdu-15","cdu-16","oauth2","jwt","multi-tenancy","spring-security","tr30"],"dg-note-properties":{"title":"Sicurezza CDU-15-16 — Modello Autorizzazione per Ente","aliases":["Sicurezza CDU-15-16 — Modello Autorizzazione per Ente"],"type":"concept","tags":["sicurezza","cdu-15","cdu-16","oauth2","jwt","multi-tenancy","spring-security","tr30"],"created":"2026-05-14","updated":"2026-07-20","sources":["2026-03-02-conspref-srs-v1-revised","2026-03-02-domande-srs-csi-v02"],"related":["[[wiki/analyses/analysis-2026-05-06-openapi-cdu-15-16\|analysis-2026-05-06-openapi-cdu-15-16]]","[[Sistemi Esterni Integrati]]","[[Architettura IaaS]]","[[CSI Piemonte]]","[[wiki/analyses/analysis-2026-05-14-risposte-mf-srs-v3\|analysis-2026-05-14-risposte-mf-srs-v3]]","[[Gestione Consensi - Applicativo]]"]}}
---


# Sicurezza CDU-15/16 — Modello Autorizzazione per Ente

> ✅ **Aggiornamento call CSI 20/07/2026 — punti sicurezza chiusi.** CSI ha confermato:
> - **Q1 (header/claim) — RISOLTO:** l'**API Manager APIMBBONE inoltra sempre al backend il Codice Fiscale (recuperato da Shibboleth/GASP) e il `codice_ente`** in header/claim. Il backend deriva l'ente da qui, non serve più validazione token lato nostro.
> - **Q3 (onboarding SIA), Q4 (TTL/refresh), Q5 (scope), Q6 (revoca credenziali): interamente DELEGATI ad APIMBBONE** (esposto in http internamente; credenziali fornite da CSI, revoca affidata a servizio di terza parte). L'onboarding di un nuovo SIA è **evento raro**, integrabile in seguito via **CR**.
> - **Conseguenza design (decisione progetto):** la tabella `cons_t_client_ente` (ex Livello B) **esce dallo scope V1.0** — il `codice_ente` autoritativo arriva dal gateway. Resta il **Livello C** (`WHERE codice_ente`) come difesa residua; la tabella è documentata come **estensione futura** per scenari multi-ente/aggregatori. Vedi §3 e §7.
>
> Le sezioni §3–§8 sotto conservano il disegno storico pre-call; i riquadri aggiornati indicano cosa resta valido.

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
> - Token e rate limiting/throttling sono **gestiti dall'APIM** → il backend può **non** validare il JWT via JWKS né implementare `bucket4j`; resta a noi l'**isolamento dei dati per ente**.
> - ✅ **Header/claim inoltrati dal gateway — confermati (call CSI 20/07/2026):** l'APIM inoltra al backend il **Codice Fiscale** (recuperato da **Shibboleth/GASP**) e il **`codice_ente`**. Il backend usa il `codice_ente` del gateway come sorgente autoritativa per l'isolamento per ente (niente lookup su `cons_t_client_ente` in V1.0) e il CF come soggetto della query. `client_id` resta disponibile lato APIM per audit/onboarding.
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
| Autorizzazione granulare **per ente** | Backend applicativo | Filter custom `EnteAuthorizationFilter` su `codice_ente` inoltrato dal Gateway (`cons_t_client_ente` fuori scope V1.0) |
| TLS termination | Piattaforma / APIM Gateway | Gateway APIM + layer rete IaaS CSI/Nivola |
| WAF | Piattaforma | Layer di rete CSI/Nivola |
| Audit log applicativo | Backend applicativo | Logger applicativo strutturato JSON |

---

## 2. Flusso end-to-end (SIA ASR → Gestione Consensi)

> Nota (aggiornata call CSI 20/07/2026): token rilasciato/validato dall'**API Manager APIMBBONE** (Key Manager + Gateway). Il backend riceve la richiesta **già autenticata dal gateway**, con **CF (da Shibboleth) e `codice_ente` inoltrati in header/claim**, e si concentra sull'isolamento per ente derivandolo dal `codice_ente` del gateway. La validazione JWT via JWKS lato backend **non è necessaria**. Cfr. §1.4.

```
[SIA ASR x]
   │ 1. POST /token  (grant_type=client_credentials, client_id, client_secret)
   ▼
[API Manager APIMBBONE — Key Manager]  ──► access token OAuth2 (client_credentials)
   │
   │ 2. GET /api/v1/...  +  Header: Authorization: Bearer <token>
   ▼
[API Gateway APIMBBONE]  ──► valida token, rate limiting/throttling, instrada al backend
   │        inoltra header/claim: Codice Fiscale (da Shibboleth) + codice_ente
   ▼
[Spring Boot Backend Gestione Consensi]
   ├─ A) Gateway trust boundary (call CSI 20/07/2026)
   │     ├─ Token già validato dall'APIM (Key Manager) — no JWKS lato backend
   │     └─ Estrazione da header/claim: codice_ente (autoritativo) + CF (soggetto)
   ├─ B) EnteAuthorizationFilter (custom, semplificato)
   │     ├─ codice_ente = header del gateway (NO lookup cons_t_client_ente in V1.0)
   │     ├─ Estrazione codice_ente eventuale dalla request (path/query)
   │     └─ Confronto header vs request: mismatch → 403 Forbidden
   ├─ C) Business logic (RestController CDU-15/16)
   │     └─ Repository query SEMPRE con WHERE codice_ente = :authorizedEnte
   │        (preso dall'header del gateway, NON dal parametro request)
   └─ D) Response 200 / 401 / 403 / 404 / 500
```

---

> ⚠️ **Inquadramento (aggiornato call CSI 20/07/2026) delle sezioni §3–§8.** Le sezioni seguenti descrivono il disegno di sicurezza **pre-APIM**. Con il modello **API Manager APIMBBONE confermato** (§1.4) e i chiarimenti della call 20/07:
> - **Emissione/validazione token** (ex Livello A, JWKS, Authorization Server) e **rate limiting** (ex `bucket4j`) sono **a carico dell'APIM** → i riferimenti a JWKS/`bucket4j`/Authorization Server qui sotto sono **superati** e mantenuti come storico.
> - **Livello B — `cons_t_client_ente` è FUORI SCOPE V1.0:** il `codice_ente` autoritativo arriva dal **Gateway APIM** via header/claim (insieme al CF da Shibboleth). La tabella di mapping `client_id → codice_ente` è mantenuta come **estensione futura** (scenari multi-ente/aggregatori), non implementata in V1.0.
> - **Restano validi e a nostro carico** l'**isolamento per ente** tramite **Livello C** (`WHERE codice_ente = :authorizedEnte`, con ente preso dall'header del gateway) e l'**audit log applicativo**.
> - Il **testo per l'SRS** effettivamente recepito in `CONSPREF-SRS-V1.0-revised_v5/v6` §6.16 è la versione **APIM** (vedi §8, riquadro aggiornato).

## 3. Isolamento dati per ente — difesa a 3 livelli

Il requisito "vedere unicamente i propri WS/consensi" non si poggia sul trasporto ma su tre controlli sovrapposti.

### Livello A — Identità del client

> ⚠️ **Superato (call CSI 20/07/2026):** rilascio credenziali e validazione token sono a carico di **APIMBBONE**; le credenziali sono fornite da CSI e la revoca è affidata a servizio di terza parte (Q6). Niente JWKS lato backend. Contenuto sotto = storico.

- Ogni SIA ASR riceve coppia `client_id` + `client_secret` dall'AS CSI Piemonte (procedura onboarding da concordare).
- AS emette JWT firmato (RS256/ES256) con claim `client_id` immodificabile.
- La chiave pubblica AS è pubblicata via endpoint JWKS; il backend la usa per verificare la firma.
- Nessun client può spacciarsi per altro ente: rotazione/revoca gestita lato AS.

### Livello B — Mapping client_id → codice_ente (ESTENSIONE FUTURA, fuori scope V1.0)

> ⚠️ **Fuori scope V1.0 (decisione progetto, call CSI 20/07/2026):** il `codice_ente` autoritativo è **inoltrato dal Gateway APIM** in header/claim, quindi il mapping applicativo `client_id → codice_ente` **non serve in V1.0**. La tabella `cons_t_client_ente` è conservata come **estensione futura** per scenari multi-ente/aggregatori (un `client_id` che deve vedere più enti) ed è attivabile via CR. Contenuto sotto = riferimento per l'estensione futura.

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

### Livello C — Filtro query (defense in depth) — VALIDO in V1.0

> ✅ **Difesa residua a nostro carico (call CSI 20/07/2026):** con Livello B fuori scope, il Livello C è l'isolamento per ente effettivo di V1.0.

- Tutte le query repository di CDU-15/16 prendono `codice_ente` **dall'header/claim inoltrato dal Gateway APIM** (via `SecurityContext`/attributo di request), **mai dal parametro request** applicativo.
- Anche in caso di bug nel filter, la query `WHERE codice_ente = :authEnte` impedisce data leak.
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

**TTL raccomandato:** 3600 secondi (storico). ✅ **Call CSI 20/07/2026:** TTL e politica di refresh sono **gestiti internamente da APIMBBONE** con proprie policy — non a carico del progetto (Q4).

### 5.3 Pseudocodice filter Spring Security

> ⚠️ **Versione storica (pre-call 20/07/2026)** — legge `client_id` dal JWT e risolve l'ente via `mapper.resolveEnte()` (tabella `cons_t_client_ente`). In **V1.0** il filter è **semplificato**: legge il `codice_ente` **direttamente dall'header/claim inoltrato dal Gateway APIM** (nessun lookup su DB), poi confronta con l'eventuale `codice_ente` in request → 403 su mismatch. Vedi pseudocodice V1.0 sotto.

```java
// --- V1.0 (call CSI 20/07/2026): codice_ente autoritativo dall'header del Gateway APIM ---
@Component
public class EnteAuthorizationFilter extends OncePerRequestFilter {

    private static final String HDR_ENTE = "X-Codice-Ente";   // header/claim inoltrato dal gateway (nome esatto da APIM)
    private static final String HDR_CF   = "X-Codice-Fiscale"; // CF recuperato da Shibboleth/GASP

    @Override
    protected void doFilterInternal(HttpServletRequest req, HttpServletResponse res,
                                    FilterChain chain) throws ServletException, IOException {
        String authorizedEnte = req.getHeader(HDR_ENTE);
        if (authorizedEnte == null || authorizedEnte.isBlank()) {
            problem(res, 403, "ente_header_missing", "Header codice_ente assente dal gateway APIM");
            return;
        }
        String requestedEnte = extractEnteFromRequest(req);          // se presente in path/query
        if (requestedEnte != null && !authorizedEnte.equals(requestedEnte)) {
            problem(res, 403, "ente_mismatch", "Ente richiesto ≠ ente autorizzato dal gateway");
            return;
        }
        req.setAttribute("authorizedEnte", authorizedEnte);          // usato da Livello C nelle query
        chain.doFilter(req, res);
    }
}
```

<details><summary>Versione storica (con lookup <code>cons_t_client_ente</code>) — riferimento estensione futura</summary>

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

</details>

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
| G3 | §3.6 (sicurezza, se esistente) | ~~rate limiting `bucket4j`~~ → **superato:** rate limiting a carico APIM (Traffic Manager) |
| G4 | §8 Modello dati | ~~Aggiungere `cons_t_client_ente`~~ → **fuori scope V1.0:** `codice_ente` inoltrato dal Gateway APIM; tabella solo come estensione futura |
| G5 | §10 Test plan | Aggiungere caso E2E cross-tenant: client A tenta lettura ente B → atteso 403 |
| G6 | §11 Ops/Monitoring | Definire dashboard audit log con outcome 401/403/429 anomali |

### Punti da chiarire con CSI — ✅ CHIUSI (call CSI 20/07/2026)

| # | Domanda | Esito |
|---|---|---|
| Q1 | ~~Quali header/claim il Gateway inoltra al backend~~ | ✅ **RISOLTO:** l'APIM inoltra sempre **CF (da Shibboleth) + `codice_ente`**. Backend deriva l'ente dal `codice_ente` del gateway |
| Q2 | ~~Algoritmo firma JWT + URL JWKS~~ | ✅ **Chiuso:** token gestito dall'APIM (Key Manager). Nostro unico prerequisito: **swagger (OpenAPI)** CDU-15/16/17 |
| Q3 | ~~Onboarding nuovo SIA + chi popola il mapping~~ | ✅ **Delegato ad APIMBBONE** (esposto http interno). Mapping `cons_t_client_ente` fuori scope V1.0. Evento **raro** → integrabile in seguito via **CR** |
| Q4 | ~~TTL token + politica refresh~~ | ✅ **Delegato ad APIMBBONE** (policy interne) |
| Q5 | ~~Scope predefiniti CSI o liberi~~ | ✅ **Delegato ad APIMBBONE** |
| Q6 | ~~Revoca credenziali compromesse~~ | ✅ **Delegato:** credenziali fornite da CSI, revoca affidata a servizio di **terza parte** |

**Unico residuo attivo:** produrre e consegnare lo **swagger (OpenAPI)** di CDU-15/16/17 per la sottoscrizione sull'APIM (prerequisito). CDU-17 **rielaborato e confermato** dal committente (call 20/07/2026) — vedi [[wiki/concepts/alternativa-batch-03-pull\|Alternativa BATCH-03 — PULL CDU-17]].

---

## 8. Testo per SRS §6.15 e §6.16 — versione APIM (recepita in v5)

> ⚠️ **Aggiornato 07/2026.** Il riquadro sotto è la versione **APIMBBONE** effettivamente inserita in `CONSPREF-SRS-V1.0-revised_v5` §6.16. La precedente formulazione (Authorization Server + JWKS + `bucket4j`, "senza API Gateway/Manager") è **superata** ed è conservata solo nel log storico.

> **Modello di sicurezza CDU-15/16**
>
> L'accesso ai servizi CDU-15 e CDU-16 è regolato da autenticazione OAuth2 `client_credentials`, con token **emesso e validato dall'API Manager del CSI Piemonte (APIMBBONE)** (Key Manager) e servizi esposti dietro l'**API Gateway** dell'API Manager; l'autorizzazione applicativa a livelli (isolamento per ente) resta a carico del Backend.
>
> L'isolamento dei dati per ente (visibilità unicamente dei propri consensi/configurazioni) è garantito da:
>
> **(a)** token OAuth2 emesso e validato dall'API Manager CSI (APIMBBONE — Key Manager), associato al `client_id` univoco del SIA chiamante; il **Gateway APIM inoltra al backend, in header/claim, il Codice Fiscale (recuperato da Shibboleth/GASP) e il `codice_ente`** del fruitore. Il backend non gestisce la validazione JWKS;
>
> **(b)** il backend assume come **sorgente autoritativa dell'ente il `codice_ente` inoltrato dal Gateway APIM** (non un mapping applicativo): l'onboarding dei SIA e l'assegnazione delle credenziali sono gestiti da APIMBBONE. *(Estensione futura, non in V1.0: tabella `cons_t_client_ente` per scenari multi-ente/aggregatori.)*
>
> **(c)** filtro Spring Security (`EnteAuthorizationFilter`) che, quando la request contiene un `codice_ente` (path/query), lo confronta con quello inoltrato dal Gateway, rispondendo `403 Forbidden` in caso di discrepanza; il repository applica sempre `WHERE codice_ente = :authorizedEnte` (ente preso dall'header del Gateway) come difesa in profondità.
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
