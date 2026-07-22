---
{"dg-publish":true,"permalink":"/wiki/sources/2026-05-05-mermaid-architettura/","title":"Diagramma Architettura Sistema — Mermaid","tags":["architettura","mermaid","diagramma","to-be","frontend","backend"],"dg-note-properties":{"title":"Diagramma Architettura Sistema — Mermaid","aliases":["Diagramma Architettura Sistema — Mermaid"],"type":"source","tags":["architettura","mermaid","diagramma","to-be","frontend","backend"],"created":"2026-05-05","updated":"2026-06-17","sources":[],"related":["[[Architettura IaaS]]","[[Gestione Consensi - Applicativo]]","[[Sistemi Esterni Integrati]]","[[GASP Salute]]"]}}
---


# Diagramma Architettura Sistema (Mermaid.txt)

**File:** `raw/Mermaid.txt`
**Formato:** Mermaid graph TD
**Rilevanza:** Diagramma architetturale ufficiale del sistema TO-BE. Fonte visuale del contratto architetturale concordato con [[wiki/entities/csi-piemonte\|CSI Piemonte]].

---

## Contenuto del diagramma

```mermaid
graph TD
    subgraph BROWSER["Browser Utente"]
        WC["Web App (Cittadino)"]
        WO["Web App (Operatore)"]
    end

    subgraph INFRA["Cloud CSI Nivola (IaaS)"]
        subgraph FL["Frontend Layer"]
            AW["Apache Web Server"]
        end
        subgraph BL["Backend Layer (Spring Boot 3)"]
            EAF["EnteAuthorizationFilter\n(isolamento dati per ente)"]
            SGC["Servizio Gestione Consensi"]
            SC["Servizio Configurazione"]
            SNAP["Snapshot Service\n(CDU-17 PULL — confermato 20/07)"]
        end
        subgraph DL["Data Layer"]
            PG[("PostgreSQL — DBaaS Nivola")]
        end
    end

    subgraph SE["Sistemi Esterni"]
        GASP["GASP Salute\n(SPID/CIE — SAML2)"]
        AURA["AURA"]
        GD["Gestione Deleghe\n(via API-Piemonte)"]
        NOT["Notificatore\n(UNP / Deleghe)"]
        SIA["SIA ASR\n(1 azienda : n sistemi)"]
        CR["Configuratore Regionale\n(PUA / IRIDE)"]
        APIM["API Manager CSI\n(TO-BE — solo nuovi fruitori esterni)"]
    end

    WC --> AW
    WO --> AW
    AW --> SGC
    AW --> SC
    SGC --> PG
    SC --> PG
    SNAP --> PG
    SGC --> AURA
    SGC --> GD
    SGC --> NOT
    SGC --> SIA
    SIA -. "CDU-15/16/17 (REST, OAuth2 JWT)" .-> EAF
    APIM -. "nuovi fruitori esterni TO-BE" .-> EAF
    EAF --> SGC
    EAF --> SNAP
    WC -. "Autenticazione SPID/CIE (SAML2)" .-> GASP
    WO -. "Profilazione gestita da PUA / Configuratore" .-> CR

    style AW fill:#e8f4f8,stroke:#2c6e9e,color:#000
    style PG fill:#e8f4f8,stroke:#2c6e9e,color:#000
    style CR fill:#f0f0f0,stroke:#555,color:#000
    style GASP fill:#f0f0f0,stroke:#555,color:#000
    style APIM fill:#f0f0f0,stroke:#555,color:#000
    style INFRA fill:#f5f9ff,stroke:#2c6e9e
    style SE fill:#f9f9f9,stroke:#888
    style BROWSER fill:#fff,stroke:#333
```

---

## Note di interpretazione

- **Due servizi backend separati:** Servizio Gestione Consensi (logica CDU) e Servizio Configurazione (parametri/informative) — coerente con SRS §4. Vedi [[wiki/concepts/gestione-consensi-applicativo\|Gestione Consensi - Applicativo]].
- **GASP Salute** — nodo separato da CR. Il cittadino accede via **SPID/CIE** direttamente tramite [[wiki/concepts/gasp-salute\|GASP Salute]], senza transitare dal Configuratore Regionale (verbale 11/06/2026).
- **CR (Configuratore Regionale / PUA / IRIDE)** — esclusivamente per profilazione operatori. Linee tratteggiate = dipendenza esterna, non chiamata diretta applicativa.
- **SIA ASR** — relazione **1 azienda : n sistemi** (verbale 11/06/2026). Un'ASR può avere più sistemi SIA; se uno è in manutenzione, CSI interrompe l'invio verso tutti. Vedi [[wiki/concepts/batch-processes\|Processi Batch]] §Gestione Manutenzione ASR.
- **Infrastruttura [[wiki/concepts/architettura-iaas\|Architettura IaaS]]:** ambiente IaaS Nivola (non ECaaS/Kubernetes) per tutti gli ambienti (verbale 11/06/2026).

- **EnteAuthorizationFilter + Snapshot Service** — aggiunti al Backend Layer per recepire il modello di sicurezza CDU-15/16 e il CDU-17 PULL (✅ confermato dal committente 20/07/2026) del documento SRS §3.3/§6.17.
- **API Manager CSI** — nodo separato, **solo TO-BE per nuovi fruitori esterni** (verbale 11/06/2026); NON è sul percorso AS-IS Frontend→Backend.

> ✅ **Aggiornamento 2026-06-18:** il nodo "API Gateway" sul percorso AS-IS (Apache→Backend) è stato **rimosso** per allineamento all'SRS §3.2 (integrazione diretta, nessun gateway per i fruitori AS-IS; Spring Security a livello applicativo). L'esposizione via **API Manager CSI** resta come canale TO-BE per i soli nuovi fruitori esterni. Rimosso anche il riferimento all'immagine ECaaS `httpd_csi · docker-base` (modello ECaaS superato → IaaS). Restano da definire con CSI i dettagli operativi IaaS (deploy/ingress/TLS).
