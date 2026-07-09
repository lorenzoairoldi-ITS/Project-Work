# Piano di Migrazione — Full Cloud (FC)

**Data:** 25/06/2026
**Stato:** Versione 1.0 — Approvato

---

## Contesto

A seguito della fusione in **TP Group srl**:
- **Thema Consulting:** 30 dipendenti, sede Torino, infrastruttura on-premise (2 rack saturi, condizionamento al limite)
- **Pro Studio:** 20 dipendenti, sede Torino (affitto non rinnovato), modello cloud-native
- **Obiettivo:** tutti i 50 dipendenti lavoreranno nella sede Thema a Torino
- **Scelta architetturale:** Full cloud — nessuna infrastruttura on-premise, eccetto HPC

---

## Motivazione della scelta full cloud

| Vincolo fisico | Soluzione full cloud |
|---|---|
| 2 rack saturi | Nessuna apparecchiatura on-premise da installare |
| Condizionamento al limite | Zero carico termico aggiuntivo in sede |
| UPS 10 minuti | Nessun server da spegnere ordinatamente |
| Fortigate FG-60F sottodimensionato | Firewall eliminato: traffico filtrato via Cloud WAF / SASE |
| Nessuna ridondanza firewall | La sicurezza è gestita a livello cloud (WAF, DDoS, IAM) |
| QNAP consumer non enterprise | Backup gestito dal provider (replica cross-region, snapshot) |

---

## Architettura target: Full cloud

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                          TP GROUP — FULL CLOUD (v2.1)                                │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                      │
│  ┌────────────────────────────────────────────────────────────────────────────┐     │
│  │                   Microsoft 365 E5 + Entra ID Premium P2                     │     │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────┐ │     │
│  │  │ Exchange │  │  Teams   │  │ OneDrive │  │SharePoint│  │   Defender   │ │     │
│  │  │ Online   │  │          │  │          │  │          │  │   for O365   │ │     │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘  └──────────────┘ │     │
│  │  ┌──────────────────────────────────────────────────────────────────┐    │     │
│  │  │  Entra ID Premium P2 + Defender for Endpoint (tutti i laptop)   │    │     │
│  │  │  2FA / Conditional Access / Passwordless / PIM / Compliance     │    │     │
│  │  └──────────────────────────────────────────────────────────────────┘    │     │
│  └────────────────────────────────────────────────────────────────────────────┘     │
│                                                                                      │
│  ┌────────────────────────────────────────────────────────────────────────────┐     │
│  │              Azure — West Europe (primaria) + North Europe (DR)             │     │
│  │                                                                              │     │
│  │  ┌────────────┐  ┌──────────────────┐  ┌──────────┐  ┌──────────────────┐ │     │
│  │  │ VM Zimbra  │  │ VM Odoo          │  │ VM Gitea │  │ VM MediaWiki     │ │     │
│  │  │ (50 utenze)│  │ (CRM+Accounting  │  │ (Git)    │  │ (documentazione) │ │     │
│  │  │            │  │  + Timesheet)    │  │          │  │                  │ │     │
│  │  └────────────┘  │ + PostgreSQL     │  └──────────┘  └──────────────────┘ │     │
│  │                  │   Flexible       │                                      │     │
│  │                  │   Server HA      │                                      │     │
│  │                  └──────────────────┘                                      │     │
│  │                                                                              │     │
│  │  ┌────────────────────────────────────┐  ┌──────────────────────────────┐ │     │
│  │  │  Pondus — IIS + **Azure SQL MI**  │  │  **File: OneDrive/SharePoint**│ │     │
│  │  │  (Business Critical, HA nativa)   │  │  (SaaS, incluso in E5,       │ │     │
│  │  │  RTO 30 min                       │  │  zero VM da gestire)          │ │     │
│  │  └────────────────────────────────────┘  └──────────────────────────────┘ │     │
│  │                                                                              │     │
│  │  ┌──────────────────────────────────────────────────────────────────────┐ │     │
│  │  │  Azure Site Recovery (ASR) — replica continua verso North Europe     │ │     │
│  │  │  VM protette: Pondus (IIS), Odoo, Zimbra, AD/DNS                    │ │     │
│  │  │  RPO 15 min, RTO ~1h per critiche / RPO 4h, RTO 4h per standard    │ │     │
│  │  └──────────────────────────────────────────────────────────────────────┘ │     │
│  │                                                                              │     │
│  │  ┌──────────────────────────────────────────────────────────────────────┐ │     │
│  │  │  Azure Backup Vault — cross-region (West Europe → North Europe)     │ │     │
│  │  │  Retention 90gg — per VM non protette da ASR (Gitea, MediaWiki)     │ │     │
│  │  └──────────────────────────────────────────────────────────────────────┘ │     │
│  └────────────────────────────────────────────────────────────────────────────┘     │
│                                                                                      │
│  ┌────────────────────────────────────────────────────────────────────────────┐     │
│  │     HPC Cluster — Bare Metal Cloud (OVH / Hetzner) — Multi-sito            │     │
│  │                                                                              │     │
│  │  Sito primario: 12 nodi (8 esistenti + 4 nuovi) — InfiniBand 200 Gbps     │     │
│  │  Sito DR:       4 nodi (warm DR — RPO 24h, RTO 4h)                         │     │
│  │  Replica notturna rsync + GlusterFS, failover manuale tramite job scheduler │     │
│  │                                                                              │     │
│  │  VPN Site-to-Site crittografata Azure ← → OVH/Hetzner (sito primario + DR) │     │
│  │  Budget: 270.000€ + IVA (noleggio 3 anni)                                   │     │
│  └────────────────────────────────────────────────────────────────────────────┘     │
│                                                                                      │
│  ┌────────────────────────────────────────────────────────────────────────────┐     │
│  │              Security Layer (ISO 27001)                                    │     │
│  │                                                                              │     │
│  │  ┌────────────┐  ┌──────────┐  ┌──────────┐  ┌────────────┐  ┌─────────┐ │     │
│  │  │ Defender   │  │ Azure    │  │ Azure    │  │ Azure      │  │ Azure   │ │     │
│  │  │ for Cloud  │  │ WAF      │  │ DDoS     │  │ Monitor +  │  │ Key     │ │     │
│  │  │ (CSPM)     │  │ (AppGW)  │  │ Protect  │  │ Sentinel   │  │ Vault   │ │     │
│  │  └────────────┘  └──────────┘  └──────────┘  └────────────┘  └─────────┘ │     │
│  │                                                                              │     │
│  │  ┌──────────────────────────────────────────────────────────────────────┐ │     │
│  │  │  VPN Gateway (S2S per HPC dual-site + P2S per remote worker)        │ │     │
│  │  └──────────────────────────────────────────────────────────────────────┘ │     │
│  └────────────────────────────────────────────────────────────────────────────┘     │
│                                                                                      │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## Mappatura servizi: on-premise → cloud

| Servizio | Origine | Destinazione full cloud | Tipo |
|---|---|---|---|
| Email / Calendar | Zimbra (VM Thema) + Gmail (Pro Studio) | **Exchange Online** (MS365) | SaaS |
| Produttività | MS365 + LibreOffice (Thema) + Google Workspace (Pro Studio) | **MS365** unificato | SaaS |
| Video / IM | MS Teams (Thema) + Google Meet (Pro Studio) | **MS Teams** | SaaS |
| Identità | Samba4 AD (Thema) + Google Identity (Pro Studio) | **Entra ID** (Azure AD) | SaaS |
| File server | NextCloud (VM Thema) + Google Drive (Pro Studio) | **OneDrive + SharePoint** (SaaS, incluso in MS365 E5) | SaaS |
| Dev / Versioning | Gitea (VM Thema) + Bitbucket (Pro Studio) | **Gitea** su Azure VM (git mirror da Bitbucket) | IaaS |
| CRM | SugarCRM (VM Thema) + Odoo CRM (Pro Studio) | **Odoo** unificato su Azure VM | IaaS |
| Timesheet | Kimai (VM Thema) + Odoo Timesheet (Pro Studio) | **Odoo Timesheet** unificato | IaaS |
| Contabilità | Profis (on-premise Windows Thema) + Odoo Accounting (Pro Studio) | **Odoo Accounting** unificato su Azure VM | IaaS |
| Documentazione | MediaWiki (VM Thema) + MediaWiki (GCP Pro Studio) | **MediaWiki** su Azure VM (istanza unica) | IaaS |
| Pondus | IIS + MSSQL su HW fisico (Thema) | **VM IIS** + **Azure SQL Managed Instance** (HA nativa, backup automatico, patch gestite) | IaaS + PaaS |
| HPC cluster | 8 nodi fisici on-premise (Thema) | **8+8 nodi bare-metal cloud** (OVH / Hetzner) | Bare Metal |

---

## Conformità ISO 27001 — Shared Responsibility Model

| Controllo ISO 27001 | Chi lo soddisfa | Come |
|---|---|---|---|
| **Business continuity** | Provider + TP Group | Azure Availability Zones per VM + Azure Site Recovery (ASR) replica continua verso North Europe per VM critiche |
| **Disaster recovery** | Provider + TP Group | ASR (RPO 15 min critiche, 4h standard) + Azure Backup cross-region per VM non ASR + HPC warm DR site (RPO 24h) |
| **Sicurezza accessi** | TP Group | Entra ID Premium P2 + Conditional Access + 2FA + PIM (Privileged Identity Management) |
| **Cifratura storage** | Provider | Azure Storage Service Encryption (SSE), BitLocker VM, Azure Key Vault Premium (HSM) |
| **Protocolli sicuri** | TP Group | HTTPS-only, TLS 1.2+, VPN IPsec/WireGuard per accessi remoti |
| **VPN** | TP Group | Azure VPN Gateway (Site-to-Site per HPC dual-site + Point-to-Site per remote worker) |
| **Audit trail** | Provider + TP Group | Azure Monitor + Log Analytics + Sentinel (SIEM + SOAR), M365 Advanced Audit |
| **Backup off-site** | Provider | Azure Backup Vault con replica in paired region (North Europe), retention 90gg |
| **RPO/RTO definiti** | TP Group | Vedi tabella dedicata § RPO/RTO target per servizio |
| **Vulnerability assessment** | TP Group | Defender for Cloud (CSPM) + penetration test esterno certificato annuale |
| **Threat detection** | Provider + TP Group | Defender for Cloud + Defender for Office 365 (P2) + Defender for Endpoint |

---

## RPO/RTO target per servizio

| Servizio | RPO | RTO | Meccanismo | Note |
|---|---|---|---|---|
| Pondus (IIS + SQL MI) | 15 min | 30 min | ASR replica continua + SQL MI HA nativa (Business Critical) | Critico |
| Odoo (CRM+Accounting+TS) | 15 min | 1 h | ASR replica continua + PostgreSQL Flexible Server Zone-Redundant HA | Critico — incluso dato contabile |
| Zimbra (email) | 4 h | 2 h | ASR replica periodica + backup giornaliero | Standard |
| AD/DNS (Entra ID sync) | 15 min | 1 h | ASR replica continua per VM Domain Controller locale | Critico per autenticazione |
| Gitea (versioning) | 24 h | 4 h | Azure Backup cross-region + git mirror | Standard — repo su git, ri-clonabile |
| MediaWiki (documentazione) | 24 h | 8 h | Azure Backup cross-region | Standard |
| HPC — carichi di calcolo | 24 h | 4 h | Replica notturna rsync/GlusterFS verso sito DR | **Warm DR**: la replica non è continua, si perde fino a 24h di scratch |
| File utente (OneDrive/SharePoint) | < 1 min | 0 min | Gestito da Microsoft (SaaS), versioning + recycle bin + retention policy | Nessuna responsabilità TP Group |

---

## Piano migrazione — servizio per servizio

### 1. Email / Calendar — Exchange Online (MS365 E5)

| | Thema | Pro Studio |
|---|---|---|
| Attuale | Zimbra (VM Linux) | Gmail / Google Calendar |
| Target | **Exchange Online** | |

**Azioni:**
- Acquistare licenze MS365 E5 per 50 utenti
- Migrare cassette postali Zimbra → Exchange Online via Exchange Migration Wizard (IMAP) o MigrationWiz (BitTitan)
- Migrare Gmail → Exchange Online via Google Workspace Migration Tool (GWMME)
- Migrare calendari e contatti (strumento di migrazione nativo MS365)
- Configurare MX, SPF, DKIM, DMARC per i domini Thema e Pro Studio verso Exchange Online
- Mantenere forward e redirect per 30 giorni post-migrazione

**Complessità:** Media

---

### 2. Produttività — MS365 (unificato)

| | Thema | Pro Studio |
|---|---|---|
| Attuale | MS365 + LibreOffice | Google Docs / Sheets / Slides |
| Target | **MS365 E5** | |

**Azioni:**
- Assegnare licenze MS365 E5 ai 20 utenti Pro Studio (Thema già in E5)
- Esportare documenti da Google Workspace in formato Office tramite Google Takeout o strumento nativo
- Caricare su OneDrive/SharePoint con struttura condivisa
- Formazione utenti Pro Studio su Office, Teams, OneDrive

**Complessità:** Bassa

---

### 3. Video / IM — Teams (unificato)

| | Thema | Pro Studio |
|---|---|---|
| Attuale | MS Teams | Google Meet / Chat |
| Target | **MS Teams** | |

**Azioni:**
- In atto: Teams già usato da Thema
- Creare canali e team per includere i 20 nuovi utenti
- Disattivare Google Meet/Chat per dominio TP Group dopo migrazione
- Formazione su Teams per utenti Pro Studio

**Complessità:** Bassa

---

### 4. Identità — Entra ID (Azure AD)

| | Thema | Pro Studio |
|---|---|---|
| Attuale | Samba4 AD (VM Linux) | Google Identity (2FA/SSO) |
| Target | **Entra ID** (Azure Active Directory) | |

**Azioni:**
- Creare tenant Entra ID per TP Group
- Sincronizzare utenti Samba4 via Azure AD Connect (o migrazione manuale, dato il numero contenuto)
- Creare account per i 20 utenti Pro Studio direttamente in Entra ID
- Configurare Conditional Access (2FA obbligatoria, accesso condizionato per sede/device)
- Applicare policy passwordless (Windows Hello / MS Authenticator)
- Disattivare Google Identity e Samba4 AD dopo completamento

**Complessità:** Media

---

### 5. File Server — OneDrive + SharePoint (MS365 E5)

| | Thema | Pro Studio |
|---|---|---|
| Attuale | NextCloud (VM Linux) | Google Drive (~2TB/utente, ~40TB totali) |
| Target | **OneDrive** (personale) + **SharePoint** (team/dipartimenti) | |

**Motivazione della scelta:** Con MS365 E5, OneDrive e SharePoint sono servizi SaaS già inclusi, con HA nativa, backup automatico, versioning, compliance tools e integrazione con Teams/Outlook. Eliminare NextCloud evita:
- Una VM IaaS costosa e single point of failure
- ~50TB di storage Azure Files da gestire e pagare
- Manutenzione e patch del sistema operativo
- Complessità operativa di un file server self-managed

**Azioni:**
- Migrare dati NextCloud esistente (Thema) → SharePoint document libraries (struttura a reparti)
- Migrare Google Drive Pro Studio (~40TB) → OneDrive (personale) + SharePoint (condivise):
  - Dati personali (< 1TB/utente): migrazione via Microsoft Migration Manager (strumento nativo MS365)
  - Dati condivisi (project drive): rclone diretto su SharePoint con metadata mapping
  - Dati bulk (~40TB): **Azure Data Box** — spedizione fisica del disco a Microsoft che carica direttamente in SharePoint/OneDrive
- Configurare OneDrive Known Folder Move su tutti i laptop (Desktop, Documenti, Foto → OneDrive)
- Impostare policy di retention e classificazione dati in Microsoft 365 Compliance Center
- Disattivare NextCloud VM e Google Drive dopo verifica

**Complessità:** Media (semplificata rispetto a NextCloud IaaS)

---

### 6. Dev / Versioning — Gitea su Azure VM

| | Thema | Pro Studio |
|---|---|---|
| Attuale | Gitea (VM Linux) | Bitbucket (SaaS Atlassian) |
| Target | **Gitea** su Azure VM | |

**Azioni:**
- Deploy VM Linux con Gitea in Azure (serie D4s v5, 4 vCPU, 16 GB RAM)
- Clonare repository da Gitea Thema e Bitbucket Pro Studio (git clone --mirror)
- Push su nuova istanza Gitea (git push --mirror)
- Verificare integrità e permessi per ogni repository
- Configurare autenticazione via Entra ID (OAuth2)
- Disattivare account Bitbucket Pro Studio

**Complessità:** Bassa

---

### 7. CRM — Odoo su Azure VM

| | Thema | Pro Studio |
|---|---|---|
| Attuale | SugarCRM (VM Linux) | Odoo CRM (SaaS) |
| Target | **Odoo** su Azure VM | |

**Azioni:**
- Deploy VM Odoo in Azure (serie E8s v5, 8 vCPU, 64 GB RAM, Availability Set per HA)
- Deploy **Azure Database for PostgreSQL Flexible Server — Zone-Redundant HA** (failover automatico < 2 min, RPO zero)
- Configurare app Odoo in multi-process (workers) per distribuire carico su 2 VM dietro Load Balancer
- Esportare dati da SugarCRM (contatti, opportunità, contratti, storico) via SugarCRM API
- Mappare campi SugarCRM → Odoo CRM
- Esportare dati CRM da Odoo SaaS Pro Studio (stessa piattaforma → merge diretto)
- Importare dati unificati in Odoo
- Verificare integrità storico commerciale
- Spegnere VM SugarCRM e disattivare account Odoo SaaS Pro Studio

**Complessità:** Alta

---

### 8. Timesheet — Odoo (unificato)

| | Thema | Pro Studio |
|---|---|---|
| Attuale | Kimai (VM Linux) | Odoo Timesheet (SaaS) |
| Target | **Odoo Timesheet** (su stessa VM Odoo CRM) | |

**Azioni:**
- Esportare dati timesheet da Kimai (CSV/API)
- Esportare dati timesheet da Odoo SaaS Pro Studio
- Importare in Odoo TP Group
- Spegnere VM Kimai dopo verifica

**Complessità:** Bassa

---

### 9. Contabilità — Odoo Accounting (unificato) — Workstream separato

| | Thema | Pro Studio |
|---|---|---|
| Attuale | Profis (on-premise, Windows) | Odoo Accounting (SaaS) |
| Target | **Odoo Accounting** unificato (stessa VM/DB Odoo di CRM) | |

> **⚠️ Attenzione:** Questa è la migrazione a rischio più alto del piano. I dati contabili hanno vincoli fiscali, normativi e di continuità con il commercialista. Richiede un **workstream dedicato** con tempistiche indipendenti dal resto della migrazione.

**Azioni:**

**Fase 1 — Preliminare (Settimana 3, parallela ad altre attività)**
- Estrarre dati da Profis: piano dei conti, saldi apertura, movimenti IVA, fatture elettroniche attive/passive, prima nota, cespiti
- Estrarre dati da Odoo SaaS Pro Studio (stessa struttura, merge più semplice)
- Coinvolgere commercialista per mappatura completa piani dei conti

**Fase 2 — Setup ambiente di staging (Settimana 4)**
- Allestire ambiente Odoo Accounting di staging su DB separato (stessa istanza PostgreSQL)
- Importare dati reali in staging (non mock)
- Eseguire **test di quadratura**: saldi contabili pre/post migrazione devono coincidere
- Ciclo di validazione con commercialista: report, saldi, partite IVA aperte

**Fase 3 — Periodo di parallelo obbligatorio (da Settimana 5, durata: 1 mese fiscale completo)**
- Tenere attivi entrambi i sistemi: Profis (sorgente ufficiale) + Odoo Accounting (affiancamento)
- Operare in doppio sul mese fiscale corrente
- Alla fine del mese: riconciliazione finale tra Profis e Odoo
- Se tutto coincide → **certificazione del commercialista** al go-live
- Se ci sono discrepanze → un altro mese di parallelo

**Fase 4 — Go-live (dopo certificazione commercialista)**
- Disattivare Profis (non prima della certificazione)
- Archiviare dati Profis in sola lettura per 10 anni (obbligo fiscale)
- Aggiornare SDI (Sistema di Interscambio) con nuovo indirizzo telematico Odoo

**Complessità:** Molto alta — isolata come workstream indipendente

---

### 10. Documentazione — MediaWiki su Azure VM

| | Thema | Pro Studio |
|---|---|---|
| Attuale | MediaWiki (VM Linux) | MediaWiki (VM GCP) |
| Target | **MediaWiki** su Azure VM (istanza unica) | |

**Azioni:**
- Deploy VM MediaWiki in Azure (serie D2s v5, 2 vCPU, 8 GB RAM)
- Esportare pagine da MediaWiki Pro Studio (XML) e da MediaWiki Thema
- Importare su MediaWiki Azure
- Riconciliare categorie, template, namespace
- Disattivare VM GCP Pro Studio

**Complessità:** Bassa

---

### 11. Pondus — Modernizzazione su Azure SQL Managed Instance

| Attuale | Target |
|---|---|
| IIS + MSSQL su HW fisico dedicato (Thema) | **VM IIS** + **Azure SQL Managed Instance** (HA nativa) |

**Azioni:**
- Eseguire P2V (Physical-to-Virtual) del server IIS con Azure Migrate verso Azure VM (serie D4s v5, 2 VM in Availability Set)
- Migrare database MSSQL ad **Azure SQL Managed Instance** (Business Critical tier, 4 vCore, HA nativa, automatic backup + point-in-time restore)
- Configurare **Azure Site Recovery** per VM IIS: replica continua verso North Europe (RPO 15 min, RTO ~1h) — in caso di outage regionale, le VM IIS vengono riavviate in North Europe
- Configurare Azure Load Balancer per IIS (Web Farm)
- Collegare Azure Application Gateway + WAF per esposizione pubblica sicura
- Test di carico con 50+ utenti
- Verificare compatibilità codice Pondus con Azure SQL MI (99% compatibile, stesso motore MSSQL)
- Documentare procedura di failover regionale e test di DR semestrale

**Complessità:** Alta

---

### 12. HPC Cluster — Bare Metal Cloud Multi-sito (OVH / Hetzner)

| Attuale | Target |
|---|---|
| 8 nodi fisici on-premise (Thema) | **16 nodi bare-metal cloud multi-sito** |

**Vincolo:** Nodi fisici obbligatori, Infiniband 100/200 Gbps, no VM/container — rispettato.

**Soluzione:** OVH High Grade o Hetzner AX series con InfiniBand su due siti distinti per DR:

| Sito | Nodi | Ruolo |
|------|------|-------|
| Primario | 12 nodi (8 migrati + 4 nuovi) | Carico produttivo HPC |
| DR (secondo sito OVH/Hetzner) | 4 nodi | **Warm DR** — failover per manutenzione programmata o guasto sito primario |

> **⚠️ RPO/RTO HPC:** Il DR HPC è di tipo **"warm"**, non "hot". La replica dei dati di calcolo (scratch, risultati) avviene via rsync notturno. In caso di failover: RPO 24h (si perdono fino a 24 ore di risultati di calcolo), RTO 4h (tempo per riallocare job sul sito DR e montare storage replicato). Questo è coerente con l'uso batch dei job HPC (simulazioni da ore/giorni), dove il checkpoint più recente è il punto di ripresa accettabile.

**Specifiche nodo (identiche su entrambi i siti):**
- 2 CPU, ≥24 core, clock elevato
- 8 GB RAM/core, no GPU
- 1,6 TB SSD scratch, Linux
- Infiniband 200 Gbps intra-sito
- VPN IPsec inter-sito + VPN verso Azure

**Budget:** 270.000€ + IVA (cloud/noleggio 3 anni — maggiorazione per multi-sito)

**Azioni:**
- Contattare OVH / Hetzner per provisioning nodi con InfiniBand su 2 datacenter distinti
- Configurare VPN IPsec/WireGuard tra Azure e datacenter HPC primario + DR
- Configurare replica dati HPC tra sito primario e DR (rsync notturno su scratch SSD + GlusterFS per storage persistente)
- Documentare RPO/RTO HPC nella procedura di DR aziendale (RPO 24h, RTO 4h)
- Migrare carichi di lavoro HPC esistenti via rsync su scratch SSD
- Documentare procedura di failover manuale (HPC batch job scheduler gestisce la coda)
- Test benchmark performance pre e post-migrazione

**Complessità:** Media

---

## Cronologia migrazione

### Fase 0 — Propedeutiche (Settimana 1-2)
- [ ] Attivare tenant MS365 E5 / Entra ID Premium P2 per TP Group
- [ ] Attivare sottoscrizione Azure con subscription dedicata
- [ ] Scrivere infrastruttura cloud come codice (Terraform + Ansible) — ambiente riproducibile
- [ ] Ordinare nodi bare-metal HPC (OVH / Hetzner) — sito primario + DR
- [ ] **Ordinare Azure Data Box** per migrazione 40TB (consegna prevista in 2-3 settimane → disponibile in Fase 2)
- [ ] Creare VPN Site-to-Site Azure ↔ OVH/Hetzner (sito primario + DR)
- [ ] Deploy Azure Site Recovery (ASR) per VM critiche con replica verso North Europe
- [ ] Deploy Azure Backup Vault e policy di replica cross-region (retention 90gg)
- [ ] Configurare Azure WAF, DDoS Protection, Defender for Cloud, Monitor + Sentinel
- [ ] Configurare budget alert Azure (massimale mensile con notifica automatica)

### Fase 1 — Identità e collaborazione (Settimana 2-3)
- [ ] Creare account Entra ID per tutti i 50 utenti (migrazione Samba4 + nuova creazione)
- [ ] Configurare Conditional Access + 2FA
- [ ] Assegnare licenze MS365 E5
- [ ] Migrare email Zimbra + Gmail → Exchange Online
- [ ] Configurare Teams per TP Group
- [ ] Configurare OneDrive Known Folder Move su tutti i laptop
- [ ] Disattivare Google Identity e Samba4 AD

### Fase 2 — Applicazioni (Settimana 3-5)
- [ ] Deploy VM Odoo (CRM + Timesheet + Accounting) con PostgreSQL Flexible Server Zone-Redundant HA
- [ ] Ricevere Azure Data Box e spedire a Microsoft per upload dati 40TB su SharePoint/OneDrive
- [ ] Migrare SugarCRM → Odoo CRM
- [ ] Migrare Kimai → Odoo Timesheet
- [ ] Deploy VM Gitea e migrare repository da Bitbucket
- [ ] Deploy VM MediaWiki e unificare wiki (Thema + Pro Studio)
- [ ] Migrare file: NextCloud (Thema) → SharePoint via rclone, in parallelo ad attesa Data Box

### Fase 2B — Workstream Contabilità (parallelo, Settimana 3-8)
- [ ] Estrarre dati Profis e mappare piano dei conti con commercialista
- [ ] Importare dati contabili in ambiente di staging Odoo
- [ ] Test di quadratura saldi (Profis pre vs Odoo staging)
- [ ] **Periodo di parallelo obbligatorio** (1 mese fiscale completo con doppia registrazione)
- [ ] Certificazione commercialista e go-live Odoo Accounting
- [ ] Archiviare Profis per obbligo fiscale 10 anni (sola lettura)

### Fase 3 — Pondus, DR test e penetration test (Settimana 5-7)
- [ ] P2V Pondus su Azure VM in Availability Set
- [ ] Migrare database MSSQL → Azure SQL Managed Instance (Business Critical)
- [ ] Configurare Azure Load Balancer + Application Gateway + WAF per Pondus
- [ ] Configurare ASR replica continua VM Pondus verso North Europe
- [ ] Test di carico Pondus con 50+ utenti
- [ ] **Eseguire penetration test esterno certificato** (con vulnerabilità ancora rimediabili sfruttando l'on-prem come fallback)
- [ ] **Finestra di remediation** (4 settimane) per criticità emerse dal pentest
- [ ] Test benchmark performance HPC su sito primario

### Fase 4 — HPC e spegnimento on-prem (Settimana 7-9)
- [ ] Ricevere e configurare nodi bare-metal cloud (sito primario + DR)
- [ ] Collegare InfiniBand e test benchmark (latenza/banda vs on-premise)
- [ ] Configurare replica notturna HPC primario → DR (rsync + GlusterFS)
- [ ] Migrare carichi HPC da nodi on-premise a cloud
- [ ] Documentare RPO/RTO HPC (24h/4h) nella procedura di DR aziendale
- [ ] Simulazione failover DR (ASR VM + HPC failover manuale)
- [ ] Disattivare account Google Workspace Pro Studio
- [ ] **Pentest finale leggero** (verifica remediation)
- [ ] Spegnere infrastruttura on-premise Thema (VMware, SAN, QNAP, FG-60F)

---

## Conformità ISO 27001 — Verifica finale

| Controllo | Come è soddisfatto |
|---|---|---|
| Business continuity | Azure ASR replica verso North Europe (Pondus, Odoo, Zimbra, AD) + HPC bare-metal multi-sito warm DR |
| Disaster recovery | ASR per VM critiche (RPO 15 min, RTO ~1h) + Azure Backup cross-region per VM standard (RPO 4h) + HPC warm DR (RPO 24h, RTO 4h) |
| Cifratura storage | Azure SSE + BitLocker + Azure Key Vault Premium (HSM) per chiavi gestite |
| VPN | Azure VPN Gateway (S2S per HPC dual-site, P2S per remote worker) |
| Controllo accessi | Entra ID Premium P2 + Conditional Access + 2FA + PIM (JIT accesso privilegiato) |
| Audit trail | Azure Monitor + Sentinel (SIEM + SOAR) + M365 Advanced Audit |
| Threat detection | Defender for Cloud (CSPM) + Defender for Office 365 (P2) + Defender for Endpoint |
| Vulnerability assessment | Defender for Cloud vulnerability scanning + penetration test esterno annuale |
| Penetration test | Eseguito in Fase 3 (con finestra di remediation) + pentest finale leggero in Fase 4 — report con evidenze per audit ISO 27001 |

---

## Rischi principali

| Rischio | Probabilità | Impatto | Mitigazione |
|---|---|---|---|---|
| Volume migrazione 40TB Google Drive → OneDrive/SharePoint | Alta | Alto | Azure Data Box per spedizione fisica (ordinato in Fase 0, disponibile in Fase 2) — migrazione a lotti per dati non bulk |
| Compliance dati HPC su cloud bare-metal | Media | Alto | Verificare data residency (OVH/Hetzner datacenter EU), VPN cifrata, clausole GDPR contrattuali |
| Resistenza utenti cambio strumenti (Google→MS365) | Media | Medio | Formazione one-to-one, affiancamento post-migrazione, periodo di transizione, Known Folder Move per transizione trasparente |
| Migrazione contabile Profis→Odoo | Alta | Alto | **Workstream separato** con periodo di parallelo obbligatorio (1 mese fiscale) e certificazione del commercialista prima del go-live |
| Latenza HPC su cloud vs on-premise | Bassa | Medio | Test benchmark pre-migrazione, OVH/Hetzner con InfiniBand dedicato |
| Costi cloud imprevisti | Media | Medio | Azure Budget + alerts, tag per chargeback, right-sizing periodico, Terraform per controllo costi |
| Compatibilità Pondus con SQL Managed Instance | Bassa | Alto | Test di compatibilità pre-migrazione (Azure Data Migration Assistant), rollback plan su SQL VM |
| Outage regione Azure West Europe | Molto bassa | Molto alto | ASR replica continua verso North Europe per VM critiche (RPO 15 min, RTO ~1h) — le VM non ASR (Gitea, MW) hanno restore da backup con RTO 4-8h |
| Vulnerabilità critiche emerse dal penetration test | Media | Alto | Pentest programmato in Fase 3 con finestra di remediation di 4 settimane + pentest finale leggero in Fase 4 — l'on-prem è ancora attivo come fallback |

---

## Stima economica preliminare

| Voce | Costo stimato |
|---|---|
| **Licenze SaaS** | |
| MS365 E5 (50 utenze × 12 mesi) | ~55€/utente/mese × 50 = 33.000€/anno |
| **Azure IaaS / PaaS** | |
| VM (Zimbra, Odoo, Gitea, MediaWiki, IIS Pondus) — NextCloud eliminato, dati su OneDrive/SharePoint | ~1.800-2.500€/mese |
| Azure SQL Managed Instance (Pondus, Business Critical 4 vCore) | ~1.500-2.000€/mese |
| Azure Database for PostgreSQL Flexible Server Zone-Redundant HA | ~400-600€/mese |
| Azure Storage (dischi VM + SQL MI backup) | ~300-500€/mese |
| Azure Site Recovery (replica continua VM critiche verso North Europe) | ~200-300€/mese |
| Azure Backup + replica cross-region + retention 90gg | ~300-500€/mese |
| **Security** | |
| Azure WAF + DDoS + Sentinel + Defender for Cloud | ~800-1.200€/mese |
| Azure VPN Gateway (S2S + P2S) | ~200-400€/mese |
| **HPC** | |
| HPC bare-metal cloud multi-sito (16 nodi, noleggio 3 anni) | 270.000€ |
| **Servizi professionali** | |
| Migrazione (sistemisti + PM, ~75 giorni/uomo) | ~40.000€ |
| IaC automation (Terraform + Ansible, 20 giorni) | ~10.000€ |
| Penetration test esterno certificato | ~12.000€ |
| **Formazione & Change Management** | |
| Formazione utenti (sessioni one-to-one + affiancamento) | ~8.000€ |
| **TOTALE primo anno** | **~387.000-410.000€** |
| **Buffer imprevisti (10%)** | **~40.000€** |
| **TOTALE COMPLESSIVO** | **~427.000-450.000€** |
| **TOTALE ricorrente annuo (dal 2° anno, senza HPC)** | **~42.000-55.000€** |

---

## Note finali

1. **HPC:** I nodi HPC rimangono fisici (vincolo rispettato) ma su cloud bare-metal multi-sito, eliminando i problemi di spazio, raffreddamento e UPS della sede Thema. Il DR è di tipo **warm** (RPO 24h, RTO 4h) — la replica notturna è adeguata per carichi batch, ma va documentata come tale per non creare aspettative non soddisfatte in audit ISO 27001
2. **Pondus su SQL Managed Instance + ASR:** si passa da SQL Server su VM (patch, backup, HA da gestire) a PaaS gestito con HA nativa, backup automatici e retention 90gg. Azure Site Recovery garantisce failover regionale verso North Europe in caso di outage West Europe (RPO 15 min, RTO ~1h)
3. **Niente NextCloud:** Tutti i file migrano su OneDrive/SharePoint, eliminando una VM IaaS costosa (E32as), 50TB di Azure Files e un SPOF architetturale. OneDrive è già in MS365 E5, con HA nativa Microsoft e backup automatico
4. **Odoo con PostgreSQL HA:** Il database PostgreSQL di Odoo (CRM + Accounting + Timesheet) usa Azure Database for PostgreSQL Flexible Server Zone-Redundant HA, con failover automatico < 2 min — nessun SPOF per i dati contabili e CRM
5. **Sede Thema:** rimane attiva come sito di lavoro, ma senza server — solo connettività Internet, switch di rete, WiFi e laptop degli utenti
6. **Laptop:** ogni utente ha un laptop aziendale + OneDrive sync per lavoro offline; nessun VDI
7. **ISO 27001:** larga parte dei controlli è delegata al provider (Azure/MS365 certificati ISO 27001), riducendo l'onere di compliance interno; penetration test esterno in Fase 3 (con remediation window) e Defender for Cloud completano il quadro
8. **Uscita da Google Workspace:** dopo la migrazione completa, si disattiva l'account Google Workspace di Pro Studio, eliminando il costo licenze duali
9. **IaC (Terraform + Ansible):** l'intera infrastruttura cloud è definita come codice — riproducibile, versionabile, rollback rapido in caso di problemi
10. **Workstream contabilità separato:** la migrazione Profis→Odoo ha un workstream indipendente con periodo di parallelo obbligatorio di 1 mese fiscale e certificazione del commercialista prima del go-live

---

*Documento basato sulle analisi AS-IS di Thema Consulting e Pro Studio, adattato per architettura full cloud.*
