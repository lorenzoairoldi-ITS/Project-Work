# TP Group srl — Proposta di Integrazione IT

**5Stack** | 09 Luglio 2026
**Team:** Lorenzo Airoldi (PM), Jacopo Merlo (Infra & HPC), Emanuele Sole (Net & Security), Gabriele Mattiolo (Cloud & App), Yassine Suriano (Doc & Integration)

<!-- SLIDE 1: Copertina + Contesto -->

---

## Slide 1 — Copertina

| Dato | Valore |
|---|---|
| **Progetto** | Integrazione IT — Thema Consulting + Pro Studio → TP Group srl |
| **Team** | 5Stack — 5 persone |
| **Modello** | Full Cloud + Zero Trust a 9 strati |
| **Budget Anno 1** | €570.000 – €628.000 |
| **Durata** | 12 settimane |
| **Dipendenti** | 50 (30 Thema + 20 Pro Studio) |

### II Progetto in 3 Righe

Due aziende del settore automotive/aerospace si fondono:
- **Thema Consulting**: 30 pax, on-premise, 2 rack saturi, HPC 8 nodi, app Pondus
- **Pro Studio**: 20 pax, cloud-native, zero infrastruttura IT

Obiettivo: unificare i 50 dipendenti nella sede Thema (Torino) con un'infrastruttura Full Cloud + Zero Trust conforme ISO 27001.

<!-- SLIDE 2: Criticita + Architettura Target -->

---

## Slide 2 — Problemi Risolti + Architettura Target

### 10 Criticita Emerse

| # | Criticita | Soluzione |
|---|---|---|
| 1 | Rack saturi, condizionamento al limite, UPS 10 min | Zero HW on-prem — tutto su cloud |
| 2 | FortiGate FG-60F sottodimensionato, nessuna HA | FortiGate 100F HA pair + Azure Firewall |
| 3 | Backup solo on-prem (QNAP) | Azure Backup + Site Recovery multi-region |
| 4 | Pro Studio: zero backup, zero firewall | Backup off-site + sicurezza perimetrale |
| 5 | Samba4 AD: no MFA, no audit | Entra ID Premium P2 |
| 6 | Nessuna protezione endpoint | Microsoft Defender for Endpoint |
| 7 | Nessun DLP/CASB | Microsoft Purview + Defender for Cloud Apps |
| 8 | Nessun audit logging | Azure Monitor + Log Analytics + Purview |
| 9 | HPC vincolato on-prem (8 nodi) | 16 nodi Azure HBv4 elastici |
| 10 | Pondus su HW fisico non ridondato | VM IIS + Azure SQL MI geo-replication |

### Architettura Target

```
M365 E5 (SaaS)          Azure West Europe          Sede Torino
  Exchange Online     +-- VM Pondus (SQL MI)    +-- FortiGate 100F HA
  Teams               +-- VM Odoo (PostgreSQL)  +-- VLAN segmentazione
  SharePoint/OneDrive +-- VM Gitea              +-- VPN S2S verso Azure
  Defender for O365   +-- VM MediaWiki          +-- VPN HPC dedicata
  Purview Compliance  +-- Azure Firewall        +-- Client VPN remote
                      +-- Backup + Site Recovery
  Entra ID P2         +-- Log Analytics + Sentinel
  (MFA + CA + PIM)    +-- Key Vault + Bastion

Azure HPC (HBv4) — 16 nodi elastici (spot + on-demand), CycleCloud + Slurm
```

<!-- SLIDE 3: Infrastruttura Cloud (Azure) -->

---

## Slide 3 — Infrastruttura Cloud (Dettaglio)

| Risorsa | Specifica | Ruolo |
|---|---|---|
| **VM Pondus** (×2) | D4s v5, 4 vCPU, 16 GB RAM, IIS + Load Balancer | App critica |
| **VM Odoo** | E8s v5, 8 vCPU, 64 GB RAM, Ubuntu | ERP (CRM + Accounting + TS) |
| **VM Gitea** | D4s v5, 4 vCPU, 16 GB RAM, Linux | Git/Versioning |
| **VM MediaWiki** | D2s v5, 2 vCPU, 8 GB RAM, Linux | Documentazione |
| **VM AD/DNS** | D2s v5, 2 vCPU, 8 GB RAM | Domain Controller |
| **SQL Managed Instance** | Business Critical, 4 vCore, geo-replication | DB Pondus |
| **PostgreSQL Flexible Server** | Zone-Redundant HA, failover <2 min | DB Odoo |
| **Azure Firewall** | Standard + NSG + DDoS Protection | Sicurezza rete |
| **Azure VPN Gateway** | S2S + P2S, con tunnel HPC dedicato | Connettivita |
| **Azure Bastion** | Accesso JIT sicuro alle VM | Management |
| **Key Vault** | Segreti, certificati, chiavi di crittografia | Secrets |
| **Azure Backup** | Cross-region, retention 90gg | Backup VM |
| **Site Recovery** | Replica verso North Europe | DR |
| **Log Analytics** | Log centralizzato + alerting | Monitoring |
| **Sentinel** (opzionale) | SIEM + SOAR | Security |

<!-- SLIDE 4: Cluster HPC -->

---

## Slide 4 — Cluster HPC

| Parametro | Specifica |
|---|---|
| **Provider** | Microsoft Azure (istanze HBv4) |
| **Nodi** | 16 elastici (spot per batch, on-demand per produzione) |
| **CPU** | 2 CPU AMD EPYC, 24+ core cad. |
| **RAM** | 8 GB/core × 48 core = 384 GB/nodo |
| **Scratch** | 1.6 TB SSD NVMe locale/nodo |
| **GPU** | Nessuna |
| **Rete** | InfiniBand HDR 200 Gbps intra-cluster |
| **OS** | Linux (CentOS / RHEL) |
| **Orchestrazione** | Azure CycleCloud + Slurm |
| **Storage** | Azure NetApp Files / Managed Lustre |

### Breakdown Costo HPC Anno 1: €230.000

| Componente | Calcolo | Importo |
|---|---|---|
| Compute on-demand (30% carico) | 16 × 1.500h × ~€4,80/h | €115.200 |
| Compute spot (70% carico) | 16 × 3.500h × ~€1,45/h | €81.200 |
| Storage HPC (NetApp 50TB) | 50TB × ~€0,45/GB/mese × 12 | €27.000 |
| Egress (50TB/anno) | 50TB × ~€0,20/GB | €10.000 |
| Setup iniziale + benchmark | Configurazione ambiente | €26.600 |

**Anni successivi:** variabile €35k–€125k/anno, nessun vincolo contrattuale.

<!-- SLIDE 5: Sicurezza - Sede + Zero Trust -->

---

## Slide 5 — Sicurezza: Sede + Zero Trust

### Perimetro Sede

| Componente | Specifica |
|---|---|
| **Firewall** | FortiGate 100F HA pair (cluster attivo-passivo) |
| Throughput FW | 10 Gbps |
| Throughput VPN | 1 Gbps |
| VPN S2S | IPsec AES-256 verso Azure |
| VPN HPC | Tunnel dedicato con QoS prioritario |
| VPN Client | SSL VPN (FortiClient) per remote worker |
| VLAN | R&D (10), Admin (20), Operativo (30), DMZ (50), Ospiti (99) |

### Zero Trust a 9 Strati

| # | Strato | Soluzione |
|---|---|---|
| 1 | **Identity** | Entra ID P2, MFA, Conditional Access, PIM |
| 2 | **Endpoint** | Defender for Endpoint (EDR + ASR) |
| 3 | **SaaS (M365)** | M365 Defender, DLP, CASB |
| 4 | **IaaS (Azure)** | Azure Firewall, NSG, JIT VM, Disk Encryption |
| 5 | **Sede** | FortiGate 100F HA, VLAN, VPN S2S |
| 6 | **Backup & DR** | Azure Backup, Site Recovery, geo-replication |
| 7 | **Audit & Compliance** | Log Analytics, Purview |
| 8 | **Governance** | Azure Policy, RBAC, PIM |
| 9 | **Training** | Attack Simulation, security awareness |

<!-- SLIDE 6: Backup & Disaster Recovery -->

---

## Slide 6 — Backup & Disaster Recovery

### RPO / RTO per Servizio

| Servizio | RPO | RTO | Meccanismo |
|---|---|---|---|
| Pondus (SQL MI) | 15 min | 30 min | Geo-replication + Site Recovery |
| Odoo (PostgreSQL) | 15 min | 1 h | Backup frequente + Site Recovery |
| Posta elettronica | 4 h | 2 h | SaaS Microsoft (SLA) |
| Entra ID / DNS | 15 min | 1 h | Ridondanza Microsoft |
| Gitea | 24 h | 4 h | Backup giornaliero + restore |
| MediaWiki | 24 h | 8 h | Backup giornaliero + restore |
| HPC | 24 h | 4 h | Failover su seconda region |
| OneDrive/SharePoint | <1 min | 0 min | SaaS Microsoft (SLA) |

### Retention Policy

| Tipo dato | Retention | Destinazione |
|---|---|---|
| VM critiche | 90 giorni | Azure Backup Vault cross-region |
| VM standard | 30 giorni | Azure Backup LRS |
| SQL MI backup | 35 giorni (PITR) + 1 anno (LTR) | Storage geo-redundant |
| Log audit | 7 anni | Log Analytics + Archive |
| Email | 7 anni (legal hold) | Purview |

<!-- SLIDE 7: Applicazioni Migrate -->

---

## Slide 7 — Applicazioni Migrate

| Servizio | Origine Thema | Origine Pro Studio | Destinazione |
|---|---|---|---|
| **Email** | Zimbra (VM) | Gmail | Exchange Online |
| **Produttivita** | MS365 + LibreOffice | Google Docs | M365 E5 |
| **Video/IM** | MS Teams | Google Meet | MS Teams |
| **File server** | NextCloud (VM) | Google Drive | OneDrive + SharePoint |
| **CRM** | SugarCRM (VM) | Odoo CRM (SaaS) | Odoo su Azure VM |
| **Timesheet** | Kimai (VM) | Odoo Timesheet (SaaS) | Odoo su Azure VM |
| **Contabilita** | Profis (Windows) | Odoo Accounting (SaaS) | Odoo su Azure VM |
| **Versioning** | Gitea (VM) | Bitbucket (SaaS) | Gitea su Azure VM |
| **Documentazione** | MediaWiki (VM) | MediaWiki (GCP) | MediaWiki su Azure VM |
| **App critica** | Pondus (HW fisico) | N/A | VM IIS + SQL MI |
| **Identity** | Samba4 AD | Google Identity | Entra ID P2 |
| **Backup** | QNAP (on-prem) | Nessuno | Azure Backup + Site Recovery |
| **Firewall** | FortiGate FG-60F | Nessuno | FortiGate 100F HA + Azure Firewall |

### Migrazione Contabile (Profis → Odoo)

Periodo di parallelo obbligatorio di 1 mese fiscale con certificazione del commercialista prima del go-live.

<!-- SLIDE 8: Roadmap 12 Settimane -->

---

## Slide 8 — Roadmap 12 Settimane

```
Settimana    1   2   3   4   5   6   7   8   9   10  11  12
Fase 1 (Prep)    ██  ██
Fase 2 (Identity)    ██  ██
Fase 3 (Collab)          ██  ██  ██
Fase 4 (App)                 ██  ██  ██  ██  ██
Fase 5 (HPC)            ██  ██  ██  ██  ██
Fase 6 (Security)                    ████████████████
Fase 7 (Governance)                          ██  ██
Fase 8 (Collaudo)                               ██  ██  ██
Fase 9 (Chiusura)                                       ██  ██
```

| Fase | Cosa si fa |
|---|---|
| **1** | Assessment, contratti Microsoft, nomina security team |
| **2** | Entra ID P2, sync Samba4 AD + Google Identity, MFA |
| **3** | Migrazione posta (Zimbra + Gmail → Exchange), file su SharePoint/OneDrive |
| **4** | Pondus su SQL MI, Odoo su VM + PostgreSQL, Gitea, MediaWiki |
| **5** | Provisioning Azure HBv4, CycleCloud + Slurm, VPN HPC dedicata |
| **6** | FortiGate HA, Defender, Backup, DR, Log Analytics |
| **7** | Terraform IaC, Azure Policy, RBAC, PIM |
| **8** | Penetration test, test RPO/RTO, formazione utenti, cut-over |
| **9** | Dismissione on-prem Thema, chiusura Google Workspace, handover |

<!-- SLIDE 9: Costi + TCO 3 Anni -->

---

## Slide 9 — Costi + TCO

### Costi Anno 1

| Voce | Importo |
|---|---|
| Infrastruttura (pass-through Azure + M365 + HW) | €450.000 |
| Costruzione 5Stack (forfait 12 settimane) | €120.000 |
| Pacchetto Standard (formazione + supporto, opzionale) | €33.000 |
| **Totale Anno 1** | **€570.000 – €603.000** |

### Ricorrenti da Anno 2

| Voce | Importo/anno |
|---|---|
| Microsoft 365 E5 (50 utenti) | €33.000 |
| Azure IaaS/PaaS (produzione) | €30.000 |
| Backup + Site Recovery | €6.000 |
| Support Plan + Bastion + Purview + Sentinel | €24.000 |
| Licenze varie | €2.000 |
| **Totale** | **~€95.000/anno** |
| HPC variabile (a consumo) | €35k–€125k/anno |

### TCO 3 Anni — Confronto

| | Full Cloud (proposta) | On-premise potenziato |
|---|---|---|
| **Anno 1** | €603.000 | €792.000 |
| **Anno 2** | €95.000 | €297.000 |
| **Anno 3** | €95.000 | €297.000 |
| **HPC variabile (3 anni)** | ~€150.000 | Incluso |
| **TOTALE 3 ANNI** | **~€943.000** | **~€1.386.000** |
| **Risparmio** | **~€443.000 (~32%)** | |

> Nota: full cloud include disaster recovery, backup off-site, compliance ISO 27001, scalabilita immediata, zero HW da gestire, team IT ridotto (6 pax vs 8).

<!-- SLIDE 10: Vantaggi + Prossimi Passi -->

---

## Slide 10 — Vantaggi & Prossimi Passi

### Vantaggi Chiave

| Vantaggio | Dettaglio |
|---|---|
| **Zero vincoli fisici** | niente rack saturi, condizionamento, UPS |
| **Sicurezza ISO 27001** | Zero Trust 9 strati, MFA, DLP, audit, DR multi-region |
| **HPC flessibile** | spot per batch, on-demand per produzione, nessun lock-in |
| **Automazione totale** | IaC (Terraform), self-healing, dashboard unificata |
| **TCO ridotto ~32%** | rispetto a potenziamento on-premise tradizionale |
| **Business Continuity** | FortiGate HA, Azure multi-region, geo-replication |
| **Team IT ridotto** | 6 risorse invece di 8 (grazie ad automazione e cloud) |

### Prossimi Passi

1. **Review congiunta** della proposta e validazione scelte architetturali
2. **Definizione contratto** e firma NDA
3. **Avvio Fase 1** — Assessment approfondito e attivazione contratti Microsoft
4. **Go-live** in 12 settimane dal via

---

*Documento generato per presentazione PowerPoint — 5Stack © 2026*
