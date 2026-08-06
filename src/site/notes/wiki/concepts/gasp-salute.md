---
{"dg-publish":true,"permalink":"/wiki/concepts/gasp-salute/","title":"GASP Salute","tags":["autenticazione","idp","spid","cie","sso","saml2","csi-piemonte"],"dg-note-properties":{"title":"GASP Salute","aliases":["GASP Salute"],"type":"concept","tags":["autenticazione","idp","spid","cie","sso","saml2","csi-piemonte"],"created":"2026-05-05","updated":"2026-07-20","sources":["2026-03-02-conspref-srs-v1-revised","2026-03-02-domande-srs-csi-v02","2026-03-12-pile-tecnologiche-csi"],"related":["[[CSI Piemonte]]","[[Gestione Consensi - Applicativo]]","[[Architettura IaaS]]","[[valutazione-qualita-srs-consensi|Valutazione Qualità SRS — Gestione Consensi]]"]}}
---


# GASP Salute

Identity Provider (IdP) federato gestito da [[wiki/entities/csi-piemonte\|CSI Piemonte]]. Fornisce autenticazione SSO per cittadini sulla piattaforma Sanità Elettronica Regione Piemonte tramite SPID e CIE.

> 🔴 **Fuori dal perimetro di sviluppo (call CSI 06/08/2026, [[wiki/docs/adr/ADR-021-perimetro-solo-operatore\|ADR-021]]):** GASP Salute serviva esclusivamente l'accesso diretto del cittadino (CDU-01b e a valle). Questo progetto sviluppa **solo la Webapp Operatore** (autenticazione PUA/RUPAR/IRIDE) — nessuna integrazione GASP Salute è da progettare in questo progetto. Pagina mantenuta come contesto storico e come riferimento se il perimetro dovesse cambiare in futuro.

---

## Ruolo nel progetto (storico — pre-ADR-021)

Tutti i CDU Cittadino (CDU-01 ÷ CDU-06) in [[wiki/concepts/gestione-consensi-applicativo\|Gestione Consensi - Applicativo]] dipendono da GASP Salute per l'autenticazione. Nessuna autenticazione diretta nell'applicativo — tutto delegato a GASP. Blocco di CDU-01 equivale a blocco dell'intera area Cittadino. **Questi CDU sono fuori scope di sviluppo dal 06/08/2026 — vedi banner sopra.**

---

## Protocollo confermato: SAML2 ✅

**Risolto — verbale CSI/Exprivia 11/06/2026.**

| Aspetto               | Stato                                                                                           |
| --------------------- | ----------------------------------------------------------------------------------------------- |
| Protocollo            | **SAML2** (confermato verbale 11/06/2026)                                                       |
| Responsabilità scelta | [[wiki/entities/csi-piemonte\|CSI Piemonte]] (referente GASP)                                                 |
| Impatto implementativo | Integrazione tramite **Shibboleth SP** (reverse-proxy CSI davanti all'app); il backend riceve l'identità già autenticata come attributi/header dal SP — **NON** `spring-security-saml2` diretto |
| Azione rimanente      | ✅ Metadata SP **ricevuti** (ambiente TEST/preprod, 07/2026). Resta: compilare il template di federazione e censire il SP presso `identita.federazione@csi.it` |
| Stato                 | ✅ Protocollo definito + metadata ricevuti — CDU-01 può essere progettato                       |

---

## Metadata Service Provider ricevuti (TEST/preprod) — 07/2026

File fornito da CSI: `preprod_metadata_tst-consprefbo-spid.isan.csi.it_gasprp_salute.xml`.

| Aspetto | Valore |
|---|---|
| Identity Provider | **GASPRP_SALUTE** — "Proxy SPID Regione Piemonte – SALUTE" (aggregatore SPID) |
| Ambiente | TEST / preprod |
| Tipo SP | **Shibboleth SP** (reverse-proxy), host `tst-consprefbo-spid.isan.csi.it` |
| EntityID | `SERVICE_PROVIDER_TST-CONSPREFBO-SPID.ISAN.CSI.IT_443_LIVx_GASPRP_SALUTE` (x = LIV1 / LIV2 / LIV3) |
| ACS (POST) | `https://tst-consprefbo-spid.isan.csi.it/tst-consprefbo-spid_443slivXgasprp_salute/Shibboleth.sso/SAML2/POST` (+ SimpleSign, Artifact) |
| Livelli autenticazione | **LIV3 SISP + LIV2 SPID**, oppure LIV3 CNS + LIV2 SPID |
| NameID | transient / persistent / unspecified |
| Federazione | inviare metadata + template a `identita.federazione@csi.it` (rif. `Template-richiesta-Federazione-Service-Provider-V02`) |

> ⚠️ **Correzione da riportare nell'SRS (§3.3.1):** l'integrazione avviene tramite **Shibboleth SP**, non `spring-security-saml2-service-provider`. Il backend Spring riceve l'identità già autenticata (attributi/header dal SP), senza implementare direttamente il flusso SAML.

---

## Come appare nell'SRS e nei documenti

- **SRS §3.2:** "L'autenticazione avverrà tramite GASP Salute a seconda del protocollo esposto (OIDC/SAML2) — dettaglio da definire con referente CSI in fase di design tecnico"
- **Domande CSI V02:** confermato che GASP Salute è l'IdP, protocollo non specificato
- **Diagramma Mermaid:** `WC -. "Autenticazione gestita da GASP Salute" .-> CR` (linea tratteggiata = dipendenza esterna)
- **Pile tecnologiche CSI:** GASP Salute non catalogato direttamente — componente infrastrutturale CSI

---

## Integrazione tecnica — SAML2 (confermato)

| Aspetto | Dettaglio |
|---|---|
| Componente SP | **Shibboleth SP** (reverse-proxy CSI) davanti all'applicativo |
| Flow | SAML2 POST binding (Shibboleth.sso), SP-initiated |
| Token | SAML Assertion XML gestita dal SP → attributi utente (incl. CF) verso l'app |
| Configurazione | metadata SP fornito + certificato; l'app legge gli attributi/header dal SP |
| Complessità | Media (il SP gestisce il protocollo; l'app consuma l'identità) |

---

## Relazione con altri sistemi di autenticazione

| Sistema             | Profilo                            | IdP                                                  |
| ------------------- | ---------------------------------- | ---------------------------------------------------- |
| GASP Salute         | Cittadino (SPID/CIE)               | Esterno a [[wiki/concepts/architettura-iaas\|Architettura IaaS]] |
| PUA / RUPAR / IRIDE | Operatore (profilo unico, call CSI 06/08/2026) | Gestito da [[wiki/entities/csi-piemonte\|CSI Piemonte]]            |
| OAuth2 `client_credentials` | SIA Aziendale (CDU-15/16/17) | **API Manager CSI (APIMBBONE)** — Key Manager (cfr. [[wiki/concepts/sicurezza-cdu-15-16\|Sicurezza CDU-15/16]] §1.4) |

> ✅ **Propagazione identità confermata (call CSI 20/07/2026).** Per le chiamate SIA CDU-15/16/17, il Gateway APIMBBONE inoltra al backend, in header/claim, il **Codice Fiscale recuperato da Shibboleth/GASP** e il **`codice_ente`** del fruitore. È il ponte tra l'autenticazione GASP (che produce il CF) e l'isolamento per ente lato backend: il backend non rifà la validazione del token. Dettaglio in [[wiki/concepts/sicurezza-cdu-15-16\|Sicurezza CDU-15/16]] §1.4/§7.

---

## ADR correlati

| ADR | Decisione |
|---|---|
| [ADR-010](ADR-010-cdu-01-split.md) | Split CDU-01 in CDU-01a Operatore + CDU-01b Cittadino (CDU-01b dipende da GASP) |
| [ADR-021](ADR-021-perimetro-solo-operatore.md) | Perimetro progetto: solo Webapp Operatore — GASP fuori scope |
