# Struttura Dettagliata — TP Group srl

**Architettura Target Full Cloud + Zero Trust**
**5Stack** — 09 Luglio 2026

---

## 1. Panoramica Generale

```
TP Group srl (50 dipendenti — Sede unica Torino)
|
+-- Microsoft 365 E5 (SaaS)
|   +-- Exchange Online (posta)
|   +-- Teams (video/IM)
|   +-- SharePoint Online (file condivisi)
|   +-- OneDrive (file personali)
|   +-- Defender for Office 365 (sicurezza SaaS)
|   +-- Purview (compliance)
|
+-- Microsoft Entra ID Premium P2 (Identity)
|   +-- MFA / Conditional Access / PIM / Identity Protection
|   +-- Sincronizzato da legacy Samba4 AD + Google Identity
|
+-- Azure West Europe (primario) + North Europe (DR)
|   +-- VNet: Pondus, Odoo, Gitea, MediaWiki, AD/DNS
|   +-- Azure SQL Managed Instance (Pondus — geo-replication)
|   +-- Azure Database for PostgreSQL Flexible Server (Odoo — Zone-Redundant HA)
|   +-- Azure Kubernetes Service (Odoo app layer)
|   +-- Azure Firewall + NSG + DDoS Protection
|   +-- Azure Backup + Site Recovery
|   +-- Azure Key Vault (segreti, certificati)
|   +-- Azure Bastion (accesso sicuro VM)
|   +-- Log Analytics + Azure Monitor (monitoring)
|   +-- Microsoft Sentinel (SIEM, opzionale)
|
+-- Azure HPC (Calcolo)
|   +-- Azure HBv4 — 16 nodi elastici (spot + on-demand)
|   +-- Azure CycleCloud + Slurm (orchestrazione)
|   +-- Azure NetApp Files / Managed Lustre (storage HPC)
|   +-- VPN HPC dedicata (FortiGate -> Azure VPN Gateway)
|
+-- Sede Torino
|   +-- FortiGate 100F HA pair (firewall ridondato)
|   +-- VPN Site-to-Site verso Azure
|   +-- VPN HPC dedicata (throughput garantito)
|   +-- VLAN segmentazione (R&D, Admin, Operativo, Ospiti)
|   +-- Switch di rete, WiFi (esistenti)
|   +-- Client VPN (Point-to-Site per remote worker)
|
+-- Endpoint
|   +-- 50 laptop aziendali (Windows 11)
|   +-- Microsoft Defender for Endpoint (EDR, ASR)
|   +-- OneDrive Known Folder Move (backup automatico desktop/documenti)
|   +-- Microsoft Authenticator (MFA passwordless)
```

---

## 2. Dettaglio Servizi

### 2.1 Identity & Access (Entra ID Premium P2)

| Componente | Descrizione |
|---|---|
| Tenant | Microsoft Entra ID (ex Azure AD) — singolo tenant TP Group |
| Sincronizzazione | Da Samba4 AD (Thema) tramite Azure AD Connect |
| Utenti cloud-only | 20 utenti Pro Studio creati direttamente in Entra ID |
| MFA | Microsoft Authenticator (push), FIDO2 security key, SMS fallback |
| Conditional Access | Device compliance + geolocalizzazione + rischi utente/accesso |
| PIM (Privileged Identity Management) | JIT accesso ruoli admin, approvazione, audit |
| Identity Protection | Risk detection, remediation automatica |
| Access Reviews | Revisione trimestrale accessi privilegiati |
| Self-service password reset | Abilitato per tutti gli utenti |
| Device join | Entra ID Join per tutti i laptop (gestione policy MDM) |

### 2.2 Collaborazione (Microsoft 365 E5)

| Servizio | Target | Origine dati | Strategia migrazione |
|---|---|---|---|
| **Exchange Online** | 50 mailbox | Zimbra (30 utenti) + Gmail (20 utenti) | MigrationWiz / GWMME |
| **Teams** | 50 utenti | Teams (Thema) + Google Meet (Pro Studio) | Creazione team/canali unificati |
| **SharePoint Online** | Siti reparto | NextCloud (dati condivisi) | Rclone + SharePoint Migration Tool |
| **OneDrive** | 50 utenti | Google Drive (dati personali) | Known Folder Move + Data Box (40TB) |
| **Yammer / Viva Engage** | Comunicazioni aziendali | Nuovo | Configurazione iniziale |
| **Viva Insights** | Benessere digitale | Nuovo | Configurazione iniziale |

### 2.3 Applicazioni — Layer IaaS/PaaS (Azure)

#### Pondus (Applicazione Critica)

| Componente | Specifica |
|---|---|
| WEB | 2 VM IIS (D4s v5) in Availability Set + Azure Load Balancer |
| DB | Azure SQL Managed Instance — Business Critical, 4 vCore, 20 GB |
| HA | SQL MI HA nativa (3 repliche sempre sincrone) |
| DR | Geo-replication verso North Europe (RPO 15 min, RTO 30 min) |
| Backup | Azure Backup + point-in-time restore (retention 90gg) |
| Accesso | Azure Application Gateway + WAF per esposizione pubblica |
| Auth | Integrazione Entra ID (OAuth2) |

#### Odoo (CRM + Accounting + Timesheet)

| Componente | Specifica |
|---|---|
| APP | AKS (Azure Kubernetes Service) — deployment containerizzato |
| DB | Azure Database for PostgreSQL Flexible Server — Zone-Redundant HA |
| Scaling | HPA (Horizontal Pod Autoscaler) su carico CPU/memoria |
| DR | Site Recovery verso North Europe (RPO 15 min, RTO 1h) |
| Storage persistenti | Azure Disk (PVC su AKS) per file attaccamenti |

#### Gitea (Git/Versioning)

| Componente | Specifica |
|---|---|
| VM | D4s v5, 4 vCPU, 16 GB RAM, Linux |
| Storage | 100 GB SSD premium |
| Backup | Azure Backup giornaliero (retention 30gg) |
| Auth | Entra ID OAuth2 |
| Repository | Mirror da Bitbucket (Pro Studio) + Gitea esistente (Thema) |

#### MediaWiki (Documentazione)

| Componente | Specifica |
|---|---|
| VM | D2s v5, 2 vCPU, 8 GB RAM, Linux |
| Storage | 50 GB SSD standard |
| Backup | Azure Backup giornaliero (retention 30gg) |
| Merge | Unione istanze Thema (on-prem) + Pro Studio (GCP) |

### 2.4 HPC (Calcolo ad Alte Prestazioni)

| Parametro | Specifica |
|---|---|
| Provider | Microsoft Azure (istanze HBv4) |
| Nodi | 16 nodi elastici (spot per batch, on-demand per produzione) |
| CPU per nodo | 2 CPU AMD EPYC, 24+ core cad. |
| RAM per nodo | 8 GB/core |
| Scratch | 1.6 TB SSD NVMe locale per nodo |
| GPU | Nessuna |
| OS | Linux (CentOS / RHEL) |
| Networking | InfiniBand HDR 200 Gbps intra-cluster (Azure HPC) |
| Orchestrazione | Azure CycleCloud + Slurm |
| Storage condiviso | Azure NetApp Files (carichi caldi) / Managed Lustre (HPC jobs) |
| Egress | Azure VPN Gateway + FortiGate VPN HPC dedicata |
| Connettivita sede | VPN HPC tunnel separato con QoS per InfiniBand over IP |

### 2.5 Sicurezza (Zero Trust a 9 Strati)

```
Strato 1  — IDENTITY     : Entra ID P2 + MFA + CA + PIM
Strato 2  — ENDPOINT     : Defender for Endpoint (EDR + ASR)
Strato 3  — SAAS (M365)  : M365 Defender + DLP + CASB
Strato 4  — IAAS (Azure) : Azure Firewall + NSG + JIT + Disk Encryption
Strato 5  — SEDE         : FortiGate 100F HA + VPN S2S + VLAN
Strato 6  — BACKUP & DR  : Azure Backup + Site Recovery + Geo-rep
Strato 7  — AUDIT        : Log Analytics + Purview Compliance
Strato 8  — GOVERNANCE   : Azure Policy + RBAC + PIM
Strato 9  — TRAINING     : Attack Simulation + Security Awareness
```

#### Dettaglio FortiGate 100F HA

| Parametro | Valore |
|---|---|
| Modello | FortiGate 100F (×2 in cluster HA attivo-passivo) |
| Throughput FW | 10 Gbps |
| Throughput VPN | 1 Gbps |
| IPS/IDS | Abilitato (categorie automotive/aerospace) |
| Web Filtering | Abilitato (controllo categorie, TLS inspection) |
| App Control | Abilitato (blocco shadow IT, social media non autorizzati) |
| VLAN | R&D (10), Admin (20), Operativo (30), Ospiti (99), DMZ (50) |
| VPN S2S | IPsec verso Azure VPN Gateway (tunnel cifrato AES-256) |
| VPN HPC | Tunnel separato con priorita di banda per carichi HPC |
| VPN Client | SSL VPN (FortiClient) per remote worker |

### 2.6 Backup & Disaster Recovery

| Servizio | RPO | RTO | Meccanismo |
|---|---|---|---|
| Pondus (SQL MI) | 15 min | 30 min | Geo-replication + Site Recovery |
| Odoo (PostgreSQL) | 15 min | 1 h | Backup frequente + Site Recovery |
| Posta elettronica | 4 h | 2 h | SaaS Microsoft (SLA) |
| Entra ID / DNS | 15 min | 1 h | Ridondanza Microsoft |
| Gitea | 24 h | 4 h | Backup giornaliero + restore |
| MediaWiki | 24 h | 8 h | Backup giornaliero + restore |
| HPC | 24 h | 4 h | Config backup + failover second region |
| OneDrive / SharePoint | <1 min | 0 min | SaaS Microsoft (SLA) |

#### Retention Policy

| Tipo dato | Retention | Destinazione |
|---|---|---|
| Backup VM critiche | 90 giorni | Azure Backup Vault (cross-region North Europe) |
| Backup VM standard | 30 giorni | Azure Backup (LRS locale) |
| SQL MI backup automatico | 35 giorni (PITR) + 1 anno (LTR) | Azure storage geo-redundant |
| Log audit | 7 anni (compliance) | Log Analytics + Archive Storage |
| Email (Exchange Online) | 7 anni (legal hold) | Purview retention policy |

### 2.7 Monitoring & Observability

| Componente | Strumento |
|---|---|
| Log centralizzato | Azure Log Analytics workspace |
| Metriche infrastruttura | Azure Monitor + Grafana dashboard |
| Alerting | Azure Monitor Alerts (email, SMS, Teams webhook) |
| SIEM (opzionale) | Microsoft Sentinel (correlazione log, SOAR) |
| Cost monitoring | Azure Cost Management + budget alert |
| Security posture | Microsoft Defender for Cloud + Secure Score |
| Vulnerability scanning | Defender for Cloud (VM + SQL MI) |

### 2.8 Automazione (IaC)

| Strumento | Utilizzo |
|---|---|
| Terraform | Provisioning completo dell'infrastruttura Azure |
| GitHub Actions | CI/CD per configurazioni Terraform |
| Self-healing | Runbook automatici su alert (Azure Automation) |
| Policy-as-Code | Azure Policy (region restriction, SKU allowed, tagging) |

---

## 3. Mappatura AS-IS / TO-BE

| Servizio | Thema (AS-IS) | Pro Studio (AS-IS) | TP Group (TO-BE) |
|---|---|---|---|
| Identity | Samba4 AD | Google Identity | **Entra ID Premium P2** |
| Email | Zimbra (VM) | Gmail | **Exchange Online** |
| Produttivita | MS365 + LibreOffice | Google Docs | **Microsoft 365 E5** |
| Video/IM | MS Teams | Google Meet | **MS Teams** |
| File server | NextCloud (VM) | Google Drive | **OneDrive + SharePoint** |
| CRM | SugarCRM (VM) | Odoo CRM (SaaS) | **Odoo su AKS** |
| Timesheet | Kimai (VM) | Odoo Timesheet (SaaS) | **Odoo su AKS** |
| Contabilita | Profis (Windows) | Odoo Accounting (SaaS) | **Odoo su AKS** |
| Versioning | Gitea (VM) | Bitbucket (SaaS) | **Gitea su Azure VM** |
| Documentazione | MediaWiki (VM) | MediaWiki (GCP) | **MediaWiki su Azure VM** |
| App critica | Pondus (HW fisico) | N/A | **VM IIS + SQL MI (Azure)** |
| HPC | 8 nodi on-prem | N/A | **16 nodi Azure HBv4** |
| Backup | QNAP (on-prem) | Nessuno | **Azure Backup + Site Recovery** |
| Firewall | FortiGate FG-60F | Nessuno | **FortiGate 100F HA + Azure Firewall** |
| Monitoring | Nessuno | Nessuno | **Azure Monitor + Grafana + Sentinel** |

---

## 4. Connettivita e Rete

### Schema Rete

```
[SEDE TORINO]
     |
     +-- [FortiGate 100F HA pair]  (Firewall perimetrale)
     |       |
     |       +-- VLAN 10 — R&D (HPC team, ingegneri)
     |       +-- VLAN 20 — Admin (finanza, HR, direzione)
     |       +-- VLAN 30 — Operativo (produzione, Pondus users)
     |       +-- VLAN 50 — DMZ (server legacy, stampanti)
     |       +-- VLAN 99 — Ospiti (WiFi isolato)
     |
     +-- VPN Site-to-Site (IPsec AES-256) ----------------> [Azure VPN Gateway]
     |                                                              |
     |                                                              +-- VNet TP-Group
     |                                                                   +-- Subnet Pondus
     |                                                                   +-- Subnet Odoo/AKS
     |                                                                   +-- Subnet Gitea/Wiki
     |                                                                   +-- Subnet AD/DNS
     |                                                                   +-- Azure Firewall (DMZ)
     |
     +-- VPN HPC Dedicata (QoS prioritario) -----------> [Azure VPN Gateway HPC]
                                                                   |
                                                                   +-- Azure HBv4 Cluster
                                                                   +-- Azure NetApp Files / Lustre
     |
     +-- VPN Client (FortiClient SSL VPN) <--- Remote Workers
```

### Indirizzamento IP (piano)

| Segmento | Range | Note |
|---|---|---|
| Sede LAN | 10.10.0.0/16 | |
| VLAN R&D | 10.10.10.0/24 | ~12 host |
| VLAN Admin | 10.10.20.0/24 | ~12 host |
| VLAN Operativo | 10.10.30.0/24 | ~18 host |
| VLAN DMZ | 10.10.50.0/24 | Stampanti, device legacy |
| VLAN Ospiti | 10.10.99.0/24 | WiFi ospiti |
| Azure VNet | 10.20.0.0/16 | West Europe primario |
| Azure DR VNet | 10.21.0.0/16 | North Europe DR |
| VPN S2S tunnel | 169.254.10.0/30 | Sede -> Azure |
| VPN HPC tunnel | 169.254.20.0/30 | Sede -> HPC |
| VPN Pool Client | 10.10.200.0/24 | Remote worker pool |

---

## 5. Costi Mensili Infrastruttura Cloud (stimati)

| Risorsa | Costo/mese stimato |
|---|---|
| VM (Pondus, Odoo, Gitea, Wiki, AD) | ~€2.000 |
| Azure SQL MI (Business Critical, 4 vCore) | ~€1.800 |
| AKS + PostgreSQL Flexible Server | ~€600 |
| Azure Firewall Standard | ~€800 |
| Azure VPN Gateway | ~€300 |
| Azure Bastion + Key Vault | ~€200 |
| Azure Backup + Site Recovery | ~€500 |
| Azure Monitor + Log Analytics | ~€300 |
| Azure NetApp Files / Lustre (HPC storage) | ~€2.250 |
| **Subtotale** | **~€8.750/mese** |
| HPC HBv4 (on-demand, 5000h/anno) | ~€19.000/mese medio |
| **TOTALE** | **~€27.750/mese (Anno 1)** |
| **TOTALE ricorrente senza HPC** | **~€8.750/mese** |

---

*Documento di struttura dettagliata — parte integrante della Proposta TP Group v1.0*
