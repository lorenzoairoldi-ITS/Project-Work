# Proposta di Integrazione IT — TP Group srl

> **Bozza v0.2** — 28 Giugno 2026  
> A cura di **5Stack**  
> Team: Lorenzo Airoldi (PM), Jacopo Merlo (Infra & HPC), Emanuele Sole (Net & Security), Gabriele Mattiolo (Cloud & App), Yassine Suriano (Doc & Integration)

---

## 1. Premessa

**5Stack** è stata incaricata di redigere un'offerta tecnico-economica per l'acquisizione e la convergenza delle infrastrutture IT di **Thema Consulting** e **Pro Studio** nella nuova entità **TP Group srl**.

Il presente documento costituisce una **bozza di proposta** volta a illustrare la visione strategica, la nuova architettura target e la roadmap di integrazione.

---

## 2. Scenario Attuale (AS-IS)

| Dimensione | Thema Consulting | Pro Studio |
|---|---|---|
| **Dipendenti** | ~30 | ~20 |
| **Sede** | Torino (immobile di proprietà) | Torino (affitto in scadenza) |
| **Modello IT** | On-premise tradizionale | Cloud-native (no IT interno) |
| **Virtualizzazione** | VMware (2 host + NetApp SAN 24TB) | N/D |
| **HPC** | 8 nodi fisici Linux + InfiniBand | N/D |
| **App. critica** | Pondus (C#/IIS + MSSQL su HW fisico) | N/D |
| **Collaborazione** | Zimbra, NextCloud, Teams | Google Workspace (Gmail, Drive, Meet) |
| **CRM / ERP** | SugarCRM, Kimai (on-prem) | Odoo SaaS (CRM, Accounting, Timesheet) |
| **Dev** | Gitea (self-hosted) | Bitbucket (SaaS) |
| **Identity** | Samba4 AD | Google Identity |
| **Backup** | QNAP 32TB (solo on-prem) | N/D |
| **Sicurezza perimetrale** | FortiGate FG-60F (singolo, no HA) | N/D |
| **Protezione endpoint** | N/D | N/D |
| **Audit & Compliance** | N/D | N/D |
| **MFA** | N/D | N/D |

### Criticità emerse

| # | Criticità | Impatto |
|---|---|---|
| 1 | **Rack saturi** — Nessuno spazio per nuovo hardware on-prem | Impossibile espandere infrastruttura fisica |
| 2 | **Raffreddamento al limite** — Impossibile aumentare densità di potenza | Rischio surriscaldamento |
| 3 | **UPS insufficiente** — Solo 10 minuti di autonomia | Spegnimenti non ordinati |
| 4 | **Nessuna ridondanza firewall** — FG-60F sottodimensionato per 50 utenti | Single point of failure, collo di bottiglia |
| 5 | **Backup solo on-premise** — Nessuna copia off-site, QNAP unico punto di guasto | Non conforme ISO 27001, perdita dati certa in caso di sinistro |
| 6 | **Pro Studio senza backup né perimetro di sicurezza** | Rischio operativo totale |
| 7 | **Samba4 AD** — Privo di funzionalità avanzate per federazione | Nessun supporto MFA, audit, Conditional Access |
| 8 | **Nessuna protezione endpoint centralizzata** | Ogni PC è autonomo, nessuna visibilità su minacce |
| 9 | **Nessun sistema DLP / CASB** | Dati sensibili non controllati, shadow IT non rilevabile |
| 10 | **Nessun audit logging strutturato** | Impossibile tracciare accessi e modifiche, non conforme ISO 27001 |

---

## 3. Nuova Struttura Proposta — TP Group srl

### 3.1 Principio guida

> **Full Cloud first**, con HPC su bare-metal cloud dedicato. **Zero Trust** come modello di sicurezza fondante.

La nuova infrastruttura unifica i 50 dipendenti in un'unica entità, operante dalla sede di Torino (immobile Thema). L'IT passa interamente in capo a TP Group, con il team 5Stack a supporto della transizione.

### 3.2 Architettura Target

| Layer | Soluzione Proposta | Dettaglio |
|---|---|---|
| **Identity & Access** | Microsoft Entra ID Premium P2 | Unifica Samba4 AD + Google Identity. Abilita: Conditional Access (geolocalizzazione, device compliance, risk), MFA (Microsoft Authenticator, FIDO2), Identity Protection (utenti e sign-in a rischio), Privileged Identity Management (JIT, approvazioni, audit), Identity Governance (access reviews, entitlement management) |
| **Email & Calendar** | Exchange Online (M365 E5) | Migrazione da Zimbra e Gmail. Protetto da Exchange Online Protection + Defender for Office 365 |
| **Produttività** | Microsoft 365 E5 | LibreOffice + Google Docs → M365 |
| **File & Collaboration** | OneDrive + SharePoint | Sostituisce NextCloud e Google Drive. Protetto da DLP (Microsoft Purview) e Defender for Cloud Apps |
| **Video & IM** | Microsoft Teams | Unifica Teams + Google Meet |
| **CRM** | Odoo (unico) su Azure VM | SugarCRM + Odoo CRM → Odoo unificato |
| **Accounting** | Odoo Accounting su Azure VM | Profis + Odoo Accounting → Odoo unificato |
| **Timesheet** | Odoo Timesheet | Kimai + Odoo TS → unificato |
| **Documentazione** | MediaWiki su Azure VM | Unifica le due istanze |
| **Dev / Git** | Gitea su Azure VM | Sostituisce Bitbucket SaaS |
| **App critica (Pondus)** | VM IIS + Azure SQL Managed Instance | Da HW fisico a cloud. SQL MI con TDE, auditing, automated backup, geo-replication |
| **HPC** | 16 nodi bare-metal cloud (OVH / Hetzner) | Da 8 nodi on-prem a 16 cloud, multi-site per DR |
| **Backup & DR** | Azure Backup + Azure Site Recovery | Backup VM e file su Azure; Site Recovery per DR site North Europe |
| **Sicurezza endpoint** | Microsoft Defender for Endpoint (incluso M365 E5) | EDR, antivirus next-gen, Attack Surface Reduction (ASR), threat hunting, automated investigation & response su tutti i device |
| **Sicurezza SaaS (M365)** | Microsoft 365 Defender | Anti-phishing (AI), Safe Attachments, Safe Links, anti-malware, DLP (Exchange, SharePoint, Teams, OneDrive), Defender for Cloud Apps (CASB: shadow IT discovery, session controls, app governance) |
| **Sicurezza rete (Azure IaaS)** | Azure Firewall + NSG + Azure DDoS Protection | Azure Firewall per DMZ e ispezione traffico; NSG per micro-segmentazione VM (Pondus, Odoo, MediaWiki, Gitea); DDoS Protection standard per VNet |
| **Sicurezza perimetrale (sede)** | FortiGate 100F (HA pair) | Upgrade da FG-60F a FG-100F in cluster HA. Gestisce: navigazione 50 utenti (web filtering, app control, IPS), VPN site-to-site con Azure, VPN HPC dedicata (con InfiniBand over IP), VLAN per reparti |
| **Segreteria & Secrets** | Azure Key Vault | Gestione centralizzata di segreti, certificati SSL/TLS, chiavi di crittografia per Pondus, Odoo, SQL MI e tutte le VM |
| **Crittografia** | Azure Disk Encryption + SQL MI TDE + TLS 1.3 | Dischi VM crittografati (BitLocker/LUKS), SQL MI con Transparent Data Encryption, tutto il traffico in transito su TLS 1.3 |
| **Audit & Compliance** | Azure Monitor + Log Analytics + Microsoft Purview | Audit log di Entra ID, M365, Azure; Log Analytics per query e alert; Purview Compliance Manager per ISO 27001; retention policy e eDiscovery; alerting su anomalie |
| **Governance** | Azure Policy + RBAC + PIM | Azure Policy per conformità (resource tagging, region restriction, allowed SKU); RBAC per privilegio minimo; PIM per accessi JIT con approvazione |
| **Vulnerability Management** | Microsoft Defender for Cloud | Security posture assessment, vulnerability scanning per VM e SQL MI, raccomandazioni di remediation, Secure Score |
| **SIEM / SOC** | Microsoft Sentinel (opzionale) | Opzione per raccolta e correlazione log da Entra ID, Azure, M365 Defender, Defender for Endpoint, FortiGate. Alert e SOAR automation |

### 3.3 RPO / RTO Target

| Servizio | RPO | RTO | Strategia |
|---|---|---|---|
| Pondus | 15 min | 30 min | SQL MI geo-replication + VM replica (Site Recovery) |
| Odoo (CRM + Accounting + Timesheet) | 15 min | 1 h | Backup frequente + Site Recovery |
| Posta elettronica | 4 h | 2 h | M365 SaaS (SLA Microsoft) |
| AD / DNS (Entra ID) | 15 min | 1 h | Entra ID (ridondanza Microsoft) |
| Gitea | 24 h | 4 h | Backup giornaliero + restore su Azure VM |
| MediaWiki | 24 h | 8 h | Backup giornaliero + restore su Azure VM |
| HPC (OVH/Hetzner) | 24 h | 4 h | Config backup + failover su sito secondario |
| OneDrive / SharePoint | < 1 min | 0 min | SaaS (SLA Microsoft) |

### 3.4 Organigramma Target (bozza)

```
┌─────────────────────────────────────┐
│          TP Group srl               │
│           (50 pax)                  │
└─────────────────────────────────────┘
                   │
     ┌─────────────┼─────────────┬──────────────┐
     │             │             │              │
┌────────────┐ ┌────────┐ ┌──────────┐ ┌──────────────┐
│   R&D      │ │Operat. │ │  Admin   │ │    IT         │
│ (HPC)      │ │(Pondus)│ │(Fin,HR)  │ │ (Management)  │
│ ~12 pax    │ │~18 pax │ │~12 pax   │ │  ~8 pax       │
└────────────┘ └────────┘ └──────────┘ └──────────────┘
                                               │
                      ┌────────────────────────┼────────────────────┐
                      │                        │                    │
               ┌─────────────┐         ┌──────────────┐    ┌──────────────┐
               │ Sysadmin    │         │ Security     │    │ Help Desk    │
               │ (Cloud+HPC) │         │ Engineer     │    │ (1st level)  │
               │ ~3 pax      │         │ ~2 pax       │    │ ~2 pax       │
               └─────────────┘         └──────────────┘    └──────────────┘
```

### 3.5 Modello di Sicurezza — Zero Trust a Strati

L'architettura di sicurezza proposta abbandona il modello tradizionale a **perimetro unico** (muro di cinta con firewall) in favore di un modello **Zero Trust** a difesa multi-livello, dove ogni strato è indipendente e ridondante.

```
┌─────────────────────────────────────────────────────────────────────┐
│                  MODELLO ZERO TRUST — TP Group                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  1. IDENTITY — Entra ID Premium P2                          │   │
│  │  ┌──────────────────────────────────────────────────────┐   │   │
│  │  │ Conditional Access (utente, device, luogo, rischio)  │   │   │
│  │  │ MFA (Authenticator / FIDO2)                          │   │   │
│  │  │ Identity Protection (risk detection, remediation)    │   │   │
│  │  │ PIM / JIT Access (privilegi temporanei + approval)   │   │   │
│  │  │ Identity Governance (access reviews, entitlement)    │   │   │
│  │  └──────────────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  2. ENDPOINT — Defender for Endpoint (M365 E5)              │   │
│  │  ┌──────────────────────────────────────────────────────┐   │   │
│  │  │ Antivirus next-gen (cloud-delivered)                 │   │   │
│  │  │ EDR (Endpoint Detection & Response)                  │   │   │
│  │  │ ASR (Attack Surface Reduction)                       │   │   │
│  │  │ Automated Investigation & Remediation                │   │   │
│  │  │ Threat Hunting (proattivo)                           │   │   │
│  │  └──────────────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  3. SAAS (M365) — Microsoft 365 Defender                   │   │
│  │  ┌──────────────────────────────────────────────────────┐   │   │
│  │  │ Exchange Online Protection (anti-spam, anti-phish)   │   │   │
│  │  │ Defender for Office 365 (Safe Attachments, Links)    │   │   │
│  │  │ DLP (Purview) su Exchange, SharePoint, Teams, OneDr. │   │   │
│  │  │ Defender for Cloud Apps (CASB) → shadow IT, session  │   │   │
│  │  │ Information Protection (AIP, sensitivity labels)     │   │   │
│  │  └──────────────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  4. IAAS (Azure) — Difesa infrastruttura cloud             │   │
│  │  ┌──────────────────────────────────────────────────────┐   │   │
│  │  │ Azure Firewall (DMZ, ispezione traffico)             │   │   │
│  │  │ NSG (micro-segmentazione: Pondus, Odoo, MediaWiki)   │   │   │
│  │  │ Azure DDoS Protection (rete)                         │   │   │
│  │  │ JIT VM Access (solo su approvazione)                 │   │   │
│  │  │ Azure Disk Encryption (BitLocker / LUKS)             │   │   │
│  │  │ Azure Key Vault (segreti, certificati, chiavi)       │   │   │
│  │  │ Defender for Cloud (vulnerability, secure score)     │   │   │
│  │  └──────────────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  5. SEDE — FortiGate (difesa perimetrale fisica)            │   │
│  │  ┌──────────────────────────────────────────────────────┐   │   │
│  │  │ FortiGate 100F HA pair (no single point of failure)  │   │   │
│  │  │ Web Filtering, App Control, IPS, Anti-malware        │   │   │
│  │  │ VPN site-to-site (Azure → Sede, crittografata)       │   │   │
│  │  │ VPN HPC dedicata (tunnel separato per InfiniBand IP) │   │   │
│  │  │ VLAN per reparti (R&D, Admin, Operativo, Ospiti)     │   │   │
│  │  └──────────────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  6. BACKUP & DR — Continuità operativa                     │   │
│  │  ┌──────────────────────────────────────────────────────┐   │   │
│  │  │ Azure Backup (VM, file, SQL MI)                     │   │   │
│  │  │ Azure Site Recovery (replica VM su North Europe)     │   │   │
│  │  │ SQL MI geo-replication (Pondus RPO 15 min)          │   │   │
│  │  │ Crittografia at-rest (Azure Storage SSE)            │   │   │
│  │  │ Crittografia in-transit (TLS 1.3 su tutto)          │   │   │
│  │  │ Retention policy (7 anni per compliance)            │   │   │
│  │  └──────────────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  7. AUDIT & COMPLIANCE — Visibilità e conformità           │   │
│  │  ┌──────────────────────────────────────────────────────┐   │   │
│  │  │ Azure Monitor + Log Analytics (log centralizzato)    │   │   │
│  │  │ Entra ID Audit Logs (sign-in, provisioning, PIM)     │   │   │
│  │  │ Microsoft Purview Compliance Manager (ISO 27001)     │   │   │
│  │  │ eDiscovery e retention policy                        │   │   │
│  │  │ Alerting su anomalie (login sospetti, privilege esc) │   │   │
│  │  │ Microsoft Sentinel (SIEM opzionale)                  │   │   │
│  │  └──────────────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  8. GOVERNANCE — Regole e controllo                        │   │
│  │  ┌──────────────────────────────────────────────────────┐   │   │
│  │  │ Azure Policy (region restriction, allowed SKU, tag)  │   │   │
│  │  │ RBAC (privilegio minimo, nessun owner permanente)    │   │   │
│  │  │ PIM (accessi privilegiati JIT con approvazione)      │   │   │
│  │  │ Access Reviews (revisione trimestrale accessi)       │   │   │
│  │  └──────────────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  9. TRAINING — Fattore umano                                │   │
│  │  ┌──────────────────────────────────────────────────────┐   │   │
│  │  │ Attack Simulation Training (phishing simulato)       │   │   │
│  │  │ Formazione su MFA, passwordless, segnalazione minacce│   │   │
│  │  │ Campagne periodiche di security awareness            │   │   │
│  │  └──────────────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

#### Perché Zero Trust?

| Modello tradizionale (AS-IS) | Zero Trust (target) |
|---|---|
| Perimetro unico: FortiGate | Perimetri multipli: Identity + Endpoint + SaaS + IaaS + Rete |
| Fiducia implicita nella rete interna | Nessuna fiducia implicita — verifica ogni richiesta |
| Nessun MFA strutturato | MFA obbligatorio su tutti gli accessi |
| Backup on-prem (unico punto di guasto) | Backup off-site + DR multi-region |
| Nessun audit logging | Audit centralizzato con retention |
| Nessuna protezione endpoint | EDR su tutti i device |
| Nessuna visibilità su shadow IT | CASB per scoprire e controllare SaaS non autorizzati |
| Nessun DLP | DLP su email, file, Teams |

---

## 4. Roadmap di Integrazione (bozza)

| Fase | Attività | Periodo stimato |
|---|---|---|
| **1. Preparazione** | Assessment approfondito, allineamento contratti M365/Azure, nomina security team | Settimana 1-2 |
| **2. Identity First** | Attivazione Entra ID, sincronizzazione Samba4 AD, provisioning utenti, configurazione MFA + Conditional Access base | Settimana 2-3 |
| **3. Migrazione Collaborazione** | Migrazione posta (Zimbra + Gmail → Exchange), file (NextCloud + GDrive → SharePoint/OneDrive), configurazione DLP + Safe Attachments/Links | Settimana 3-5 |
| **4. Migrazione Applicazioni** | Pondus su Azure VM + SQL MI, Odoo unificato su Azure VM, MediaWiki e Gitea su Azure VM. Configurazione NSG, Key Vault, Disk Encryption per ogni VM | Settimana 4-8 |
| **5. HPC** | Selezione provider (OVH/Hetzner), provisioning 16 nodi, VPN HPC dedicata, riconfigurazione ambiente | Settimana 3-6 |
| **6. Sicurezza & DR** | **Phase 6.1 — Identity & Endpoint:** Conditional Access avanzato (risk-based), PIM, Identity Protection, onboarding Defender for Endpoint su tutti i device, Attack Simulation Training | Settimana 5-8 |
| | **Phase 6.2 — Rete & Infrastruttura:** Deploy FortiGate 100F HA pair, Azure Firewall + NSG regole, Azure DDoS Protection, VPN site-to-site Azure↔Sede, VLAN configurazione, JIT VM Access, Defender for Cloud onboarding | |
| | **Phase 6.3 — Backup & DR:** Configurazione Azure Backup (VM, file, SQL MI), Azure Site Recovery (replica North Europe), SQL MI geo-replication, test RPO/RTO | |
| | **Phase 6.4 — Audit & Compliance:** Log Analytics workspace, forwarding log da Entra ID + M365 + Azure + FortiGate, configurazione Purview Compliance Manager, alerting su anomalie | |
| **7. Governance & Razionalizzazione** | Azure Policy assignment, RBAC refinement, access reviews, Purview labels + retention policy, Microsoft Sentinel setup (se previsto) | Settimana 8-9 |
| **8. Collaudo & Go-Live** | Penetration test (interno + esterno), test RPO/RTO completi, formazione utenti (security awareness + M365), cut-over finale | Settimana 9-11 |
| **9. Chiusura** | Dismissione infr. on-prem Thema, chiusura account Google Workspace, handover al team IT TP Group, documentazione as-built | Settimana 11-12 |

---

## 5. Vantaggi della Soluzione

- **Riduzione complessità IT** — Da due stack eterogenei a un unico ecosistema Microsoft + Azure
- **Risparmio sui costi** — Eliminazione manutenzione HW on-prem, razionalizzazione licenze (TCO ridotto del ~30% su orizzonte 3 anni vs mantenimento on-prem)
- **Scalabilità** — Cloud elastico, nessun vincolo fisico (rack saturi, raffreddamento)
- **Sicurezza Zero Trust a 9 strati** — Non un singolo firewall, ma difesa indipendente su ogni livello:
  - Identity (MFA, CA, PIM) → chi accede
  - Endpoint (Defender EDR) → da dove accede
  - SaaS (M365 Defender, DLP, CASB) → cosa fa sui dati
  - IaaS (Azure Firewall, NSG, JIT) → protezione carichi cloud
  - Sede (FortiGate HA) → connettività sicura ufficio
  - Backup & DR → continuità operativa
  - Audit & Compliance → visibilità totale
  - Governance → controllo e policy
  - Training → fattore umano
- **Backup & Disaster Recovery** — Off-site su Azure, DR site su North Europe, RPO/RTO definiti per ogni servizio, conformi a requisiti ISO 27001
- **Audit e Compliance** — Log centralizzato, retention policy, Microsoft Purview per ISO 27001, eDiscovery, DLP
- **Business Continuity** — Nessun single point of failure (FortiGate in HA, Azure multi-region, backup off-site)
- **HPC performante** — Passaggio da 8 a 16 nodi bare-metal cloud, multi-site per DR
- **Prevenzione proattiva** — Vulnerability scanning (Defender for Cloud), threat hunting (Defender for Endpoint), SIEM opzionale (Sentinel)
- **Formazione continua** — Attack Simulation Training, security awareness campaign

---

## 6. Budget Indicativo (Primo Anno)

| Voce | Costo stimato (€) |
|---|---|
| Licenze Microsoft 365 E5 (50 utenti) | ~33.000 |
| Azure IaaS / PaaS (VM Pondus, Odoo, MediaWiki, Gitea, SQL MI, Key Vault, NSG, etc.) | ~35.000 |
| Azure Firewall + Azure DDoS Protection + Defender for Cloud | ~10.000 |
| Azure Backup + Azure Site Recovery (DR North Europe) | ~8.000 |
| HPC bare-metal cloud (OVH / Hetzner — 3 anni) | ~270.000 |
| FortiGate 100F HA pair (HW + licenze 3 anni) | ~15.000 |
| Servizi professionali 5Stack (migrazione, setup, configurazione sicurezza, formazione) | ~45.000 |
| Penetration test (interno + esterno) | ~12.000 |
| Formazione utenti + Attack Simulation Training setup | ~8.000 |
| Microsoft Sentinel (opzionale, primo anno) | ~5.000 |
| **Subtotale** | **~441.000** |
| Buffer (10%) | ~44.000 |
| **Totale primo anno** | **~485.000 €** |

> *Nota: costi HPC e FortiGate HW ripartiti su 3 anni. Costi ricorrenti annuali (M365 + Azure + licenze) stimati in ~95.000 € dal secondo anno. Sentinel è opzionale e attivabile dal secondo anno.*

---

## 7. Conclusioni e Prossimi Passi

La presente bozza delinea la strategia proposta da **5Stack** per accompagnare **TP Group srl** nella fusione IT delle due realtà. Il modello **Full Cloud + HPC bare-metal dedicato + Sicurezza Zero Trust a 9 strati** risponde a tutte le criticità emerse e pone le basi per una crescita scalabile, sicura e conforme.

### Punti chiave della proposta

- **Unica identità digitale** per tutti i 50 dipendenti (Entra ID)
- **Unico ecosistema** M365 per collaborazione, posta, file
- **Cloud ibrido** per applicazioni e dati (Azure)
- **HPC cloud** per carichi di calcolo (OVH/Hetzner)
- **Sicurezza multistrato** senza punti ciechi
- **Continuità operativa** con DR su due regioni

### Prossimi passi suggeriti

1. **Review congiunta** della bozza e validazione delle scelte architetturali
2. **Raccolta feedback** su RPO/RTO e priorità di migrazione
3. **Definizione contratto** e avvio fase di preparazione (assessment dettagliato)

---

*Documento da considerarsi come **bozza preliminare**. Ogni sezione è suscettibile di modifica e approfondimento in base al confronto con il cliente.*
