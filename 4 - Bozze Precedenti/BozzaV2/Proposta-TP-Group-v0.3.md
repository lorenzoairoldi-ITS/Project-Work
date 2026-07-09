# Proposta di Integrazione IT — TP Group srl v0.4

**5Stack** — 06 Luglio 2026
**Infrastruttura anno 1:** €450.000 (pass-through)
**Costruzione 5Stack:** €120.000 (forfait)
**Ricorrenti fissi da anno 2:** ~€95.000/anno (+ HPC variabile a consumo)

---

## 1. Scenario AS-IS

### Thema Consulting (~30 dipendenti — On-premise)
| Componente | Dettaglio |
|---|---|
| Virtualizzazione | VMware — 2 host + NetApp SAN 24TB |
| HPC | 8 nodi fisici + InfiniBand |
| Collaborazione | Zimbra, NextCloud, Teams |
| Applicazioni | SugarCRM, Kimai, Gitea on-prem |
| Identity | Samba4 AD |
| Sicurezza | FortiGate FG-60F (no HA) |
| Backup | QNAP 32TB on-prem |
| Lacune | No MFA, no endpoint protection, no audit |

### Pro Studio (~20 dipendenti — Cloud-native)
| Componente | Dettaglio |
|---|---|
| Virtualizzazione | Nessuna (N/D) |
| HPC | Non disponibile |
| Collaborazione | Google Workspace (Gmail, Drive, Meet) |
| Applicazioni | Odoo SaaS, Bitbucket |
| Identity | Google Identity |
| Sicurezza | Nessuna perimetrale |
| Backup | Non disponibile |
| Lacune | No MFA, no endpoint protection, no audit |

---

## 2. Criticità Emerse (10 rischi)

1. Rack saturi — nessuno spazio per nuovo hardware
2. Raffreddamento al limite — rischio surriscaldamento
3. UPS insufficiente — solo 10 minuti di autonomia
4. Nessuna ridondanza firewall — FG-60F sottodimensionato
5. Backup solo on-premise — non conforme ISO 27001
6. Pro Studio senza backup né perimetro di sicurezza
7. Samba4 AD — nessun MFA, audit, Conditional Access
8. Nessuna protezione endpoint centralizzata
9. Nessun sistema DLP/CASB — shadow IT non rilevabile
10. Nessun audit logging strutturato

---

## 3. Architettura Target

```
                    ┌─────────────────────────────────────┐
                    │    ZERO TRUST — 9 STRATI            │
                    │ Identity │ Endpoint │ SaaS │ IaaS   │
                    │ Sede │ Backup │ Audit │ Gov │ Training│
                    └─────────────────────────────────────┘
                                       │
         ┌─────────────────────────────┼─────────────────────────────┐
         ▼                             ▼                             ▼
┌─────────────────────┐   ┌─────────────────────┐   ┌─────────────────────┐
│     IDENTITY        │   │   COLLABORATION      │   │   APPLICATIONS      │
│  Entra ID P2        │   │   Exchange Online    │   │   Pondus: VM +      │
│  MFA + CA + PIM    │   │   M365 Business      │   │   SQL MI (geo-rep)  │
│  Sincronia da       │   │   Premium + E3 Sec   │   │   Odoo: AKS +       │
│  Samba4 AD +        │   │   OneDrive/SPO       │   │   PostgreSQL        │
│  Google Identity    │   │   Teams              │   │   Gitea: AKS        │
│                     │   │   DLP + CASB         │   │   MediaWiki: AKS    │
└─────────────────────┘   └─────────────────────┘   └─────────────────────┘
         │                       │                             │
         └───────────────────────┼─────────────────────────────┘
                                 ▼
         ┌───────────────────────────────────────────────────────┐
         │                      HPC LAYER                        │
         │  Azure HBv4 spot (batch) + on-demand (produzione)    │
         │  Azure CycleCloud + Slurm                            │
         │  VPN HPC dedicata (FortiGate → Azure VPN Gateway)   │
         │  Azure NetApp Files / Managed Lustre per storage     │
         └───────────────────────────────────────────────────────┘
                                 │
         ┌───────────────────────┼───────────────────────────┐
         ▼                       ▼                           ▼
┌─────────────────┐   ┌─────────────────────┐   ┌─────────────────────┐
│     SEDE         │   │   BACKUP & DR        │   │   MONITORING        │
│ FortiGate 100F   │   │   Azure Backup       │   │   Azure Monitor +   │
│  HA pair         │   │   Site Recovery      │   │   Log Analytics     │
│ VPN S2S + HPC    │   │   SQL MI geo-rep    │   │   Grafana Dashboard │
│ Azure File Sync  │   │   Archive Storage    │   │   Sentinel (da m6)  │
│  + cache locale  │   │                      │   │                     │
│ Switch + VLAN    │   │                      │   │                     │
└─────────────────┘   └─────────────────────┘   └─────────────────────┘
```

---

## 4. Modello Zero Trust — 9 Strati

| # | Strato | Soluzione |
|---|--------|-----------|
| 1 | **Identity** | Entra ID Premium P2, MFA, Conditional Access, PIM, Access Reviews |
| 2 | **Endpoint** | Defender for Endpoint (EDR, ASR, threat hunting) |
| 3 | **SaaS (M365)** | M365 Defender, Anti-phishing, Safe Links/Attachments, DLP, CASB |
| 4 | **IaaS (Azure)** | Azure Firewall Basic, NSG, JIT VM Access, Key Vault, Disk Encryption |
| 5 | **Sede** | FortiGate 100F HA, VPN S2S, VLAN segmentazione |
| 6 | **Backup & DR** | Azure Backup, Site Recovery, SQL MI geo-replication |
| 7 | **Audit & Compliance** | Log Analytics, Purview, ISO 27001 |
| 8 | **Governance** | Azure Policy, RBAC, PIM, Access Reviews |
| 9 | **Training** | Attack Simulation Training, security awareness |

---

## 5. Dettaglio Architetturale

### 5.1 Identity & Access
- **Microsoft Entra ID Premium P2**
- Unifica Samba4 AD + Google Identity in unico tenant
- MFA obbligatorio su tutti gli accessi
- Conditional Access (device compliance + risk-based)
- PIM per admin (JIT + approvazione)
- Access Reviews trimestrali

### 5.2 Collaborazione & Produttività
- **Exchange Online** — migrazione da Zimbra e Gmail
- **Microsoft 365 Business Premium + E3 Security**
- **OneDrive + SharePoint Online** — sostituisce NextCloud e Google Drive
- DLP su email, file, Teams

### 5.3 Applicazioni (AKS nativo)
- **Pondus** → VM IIS + Azure SQL Managed Instance (TDE + geo-replication)
- **Odoo** → AKS + PostgreSQL flessibile, scaling automatico
- **Gitea** → AKS (container leggero)
- **MediaWiki** → AKS (container leggero)
- Nodi AKS spot per risparmio, on-demand per produzione

### 5.4 HPC
| Parametro | Valore |
|---|---|
| Provider | Azure (istanze HBv4/NDv5) |
| Nodi | Da 8 a 16 elastici |
| Modalità | Spot per batch/carichi non critici, on-demand per produzione |
| Orchestrazione | Azure CycleCloud + Slurm |
| Rete | VPN HPC dedicata (FortiGate → Azure VPN Gateway) |
| Storage | Azure NetApp Files / Managed Lustre per job HPC |
| DR | Failover su seconda region Azure |
| Costo | ~€200k (vs €270k bare-metal vincolato 3 anni) |

### 5.5 Sicurezza perimetrale — Sede
- **FortiGate 100F in HA**
- VPN Site-to-Site verso Azure
- VPN HPC dedicata (throughput garantito)
- VLAN segmentazione interna

### 5.6 Storage
| Tipo | Soluzione | Dettaglio |
|---|---|---|
| File caldi (sede) | Azure File Sync + cache locale | Latenza zero, sync automatico |
| File collaborativi | SharePoint Online | Sostituisce NextCloud + Google Drive |
| Database | Azure SQL MI, PostgreSQL flessibile | Geo-replica per Pondus |
| HPC | Azure NetApp Files / Managed Lustre | Performance carichi HPC |
| Dati freddi | Azure Archive Storage | ~€0,002/GB/mese |
| Backup | Azure Backup + Site Recovery | Off-site, retention compliant |
| Crittografia | Azure Disk Encryption + Key Vault + TLS 1.3 | At-rest e in-transit |

### 5.7 Automation & IaC
- **Terraform** per deploy completo dell'infrastruttura
- **GitHub Actions** per CI/CD delle configurazioni
- **Self-healing** su alert (auto-remediation runbook)
- **Policy-as-Code** con Azure Policy
- Dashboard **Grafana + Azure Monitor** unificato

### 5.8 RPO / RTO Target
| Servizio | RPO | RTO | Strategia |
|---|---|---|---|
| Pondus (SQL MI) | 15 min | 30 min | Geo-replication + Site Recovery |
| Odoo (PostgreSQL) | 15 min | 1 h | Backup frequente + Site Recovery |
| Posta elettronica | 4 h | 2 h | SaaS (SLA Microsoft) |
| AD / DNS (Entra ID) | 15 min | 1 h | Ridondanza Microsoft |
| Gitea | 24 h | 4 h | Backup giornaliero + restore |
| MediaWiki | 24 h | 8 h | Backup giornaliero + restore |
| HPC | 24 h | 4 h | Config backup + multi-region |
| OneDrive / SharePoint | < 1 min | 0 min | SaaS (SLA Microsoft) |

---

## 6. Roadmap (9 fasi — ~12 settimane)

```
     Sett. 1   2   3   4   5   6   7   8   9  10  11  12
1. Prep        ██  ██
2. Identity        ██  ██
3. Collab              ██  ██  ██
4. App                    ██  ██  ██  ██  ██
5. HPC               ██  ██  ██  ██  ██
6. Security                     ████████████████
7. Governance                              ██  ██
8. Collaudo                                   ██  ██  ██
9. Chiusura                                           ██  ██
```

### Fase 1 — Preparazione (Sett. 1-2)
Assessment, contratti M365/Azure, nomina security team

### Fase 2 — Identity First (Sett. 2-3)
Entra ID P2, sync Samba4 AD, MFA + Conditional Access base

### Fase 3 — Migrazione Collaborazione (Sett. 3-5)
Posta e file su M365, DLP, Safe Attachments/Links

### Fase 4 — Migrazione Applicazioni (Sett. 4-8)
Pondus, Odoo su AKS, MediaWiki, Gitea su Azure; NSG, Key Vault, SQL MI

### Fase 5 — HPC (Sett. 3-6)
Azure HBv4 spot + on-demand, VPN HPC dedicata, CycleCloud

### Fase 6 — Sicurezza & DR (Sett. 5-8, 4 sotto-fasi parallele)
- **6.1** Identity & Endpoint: CA avanzato, PIM, Defender for Endpoint
- **6.2** Rete & Infrastruttura: FortiGate 100F HA, Azure Firewall, NSG
- **6.3** Backup & DR: Azure Backup, Site Recovery, test RPO/RTO
- **6.4** Audit & Compliance: Log Analytics, Purview, Grafana dashboard

### Fase 7 — Governance & Razionalizzazione (Sett. 8-9)
Azure Policy, RBAC, Terraform IaC, GitHub Actions, self-healing

### Fase 8 — Collaudo & Go-Live (Sett. 9-11)
Penetration test, test RPO/RTO, formazione, cut-over

### Fase 9 — Chiusura (Sett. 11-12)
Dismissione on-prem Thema, handover, documentazione as-built

---

## 7. Organigramma Target

```
                    ┌─────────────────────────┐
                    │   TP Group srl          │
                    │   50 dipendenti          │
                    └─────────────────────────┘
                               │
        ┌──────────────────────┼──────────────────────┐
        ▼                      ▼                      ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ R&D (HPC)    │    │ Operations   │    │ Admin        │
│ ~12 pax      │    │ (Pondus)     │    │ (Fin, HR)    │
└──────────────┘    │ ~18 pax      │    │ ~12 pax      │
                    └──────────────┘    └──────────────┘
                    ┌─────────────────────────────────────┐
                    │  IT (Management) ~6 pax             │
                    │  Sysadmin (Cloud+HPC) ~2 pax        │
                    │  Security Engineer ~2 pax           │
                    │  Help Desk (1st level) ~2 pax       │
                    └─────────────────────────────────────┘
```

---

## 8. Budget Infrastruttura (pass-through)

Costi diretti di Azure, Microsoft 365 e hardware. 5Stack agisce come system integrator senza applicare markup su queste voci.

### Primo Anno

| Voce | Costo | Note |
|---|---|---|
| **Infrastruttura Core** | | |
| M365 E5 (50 utenti) | €33.000 | Upgrade da BP+E3 — include Purview, Defender P2, Compliance |
| Azure IaaS/PaaS (VM, SQL MI, AKS, PostgreSQL) | €40.000 | Con RI + Azure Hybrid Benefit |
| Azure Firewall Standard + NSG + DDoS Protection | €10.000 | Incluso filtro TLS e IDPS |
| Azure Backup + Site Recovery | €8.000 | |
| **HPC** | | |
| Azure HBv4 spot + on-demand (16 nodi elastici) | €230.000 | 5.000h stimate anno 1 |
| Azure NetApp Files / Managed Lustre (storage HPC) | €27.000 | Storage ad alte prestazioni per carichi HPC |
| HPC egress (dati in uscita verso sede) | €10.000 | Trasferimento output HPC |
| **Infrastruttura accessoria** | | |
| Azure Support Plan Standard | €12.000 | Risposta <1h, indispensabile per HPC in produzione |
| Azure Bastion | €2.000 | Accesso sicuro VM JIT senza IP pubblico |
| **Compliance & Governance** | | |
| Microsoft Purview | €5.000 | Data governance, compliance ISO 27001 |
| **Rete Sede** | | |
| FortiGate 100F HA pair (3 anni) | €15.000 | |
| **SIEM (opzionale, da mese 6)** | | |
| Microsoft Sentinel | €5.000 | |
| **Subtotale Infrastruttura** | **€397.000** | |
| **Buffer imprevisti (~13%)** | **~€53.000** | |
| **TOTALE INFRASTRUTTURA ANNO 1** | **~€450.000** | |

### Costi Ricorrenti Fissi (da anno 2)

| Voce | Costo/anno |
|---|---|
| M365 E5 | €33.000 |
| Azure IaaS/PaaS (produzione) | €30.000 |
| Azure Backup + Site Recovery | €6.000 |
| Azure Support Plan Standard | €12.000 |
| Azure Bastion | €2.000 |
| Microsoft Purview | €5.000 |
| Microsoft Sentinel | €5.000 |
| Licenze varie | €2.000 |
| **Totale ricorrente fisso** | **~€95.000** |

> **HPC variabile**: compute, storage ed egress sono a consumo in base all'uso effettivo — stimato complessivo €35.000–€125.000/anno

---

## 9. Servizi 5Stack — Costruzione Infrastruttura

### Il team 5Stack

| Ruolo | Tariffa/giorno | Nr. |
|---|---|---|
| Project Manager | €650 | 1 |
| Sistemista Senior | €500 | 4 |

*Trasferta: €110/giornata (rimborso km €0,80 + pasto) per presenza in sede cliente.*

### Dettaglio fasi di costruzione (12 settimane)

| Fase | PM (€650) | Sistemisti (€500) | **Importo** |
|---|---|---|---|
| **1. Preparazione** — assessment, contratti, security team | 3gg | 5gg | €4.450 |
| **2. Identity** — Entra ID P2, sync AD+Google, MFA, CA | 1gg | 9gg | €5.150 |
| **3. Collaborazione** — migrazione posta Zimbra+Gmail, DLP | 2gg | 18gg | €10.300 |
| **4. Applicazioni** — Pondus SQL MI, Odoo/AKS, Gitea, Wiki | 5gg | 35gg | €20.750 |
| **5. HPC** — HBv4, CycleCloud, VPN dedicata, Lustre | 2gg | 18gg | €10.300 |
| **6. Sicurezza & DR** — FortiGate HA, Defender, Backup, Sentinel | 3gg | 27gg | €15.450 |
| **7. Governance** — Terraform IaC, Policy, RBAC, self-healing | 2gg | 14gg | €8.300 |
| **8. Collaudo & Go-Live** — pen test, test RPO/RTO, cut-over | 5gg | 23gg | €14.750 |
| **9. Chiusura** — dismissione on-prem, as-built, handover | 3gg | 12gg | €7.950 |
| **Subtotale** | **26gg = €16.900** | **161gg = €80.500** | **€97.400** |
| Trasferta (stima 10gg presenza in sede) | | | **€1.100** |
| **TOTALE COSTRUZIONE (forfait)** | | | **€120.000** |

### Cosa include

- Progettazione architetturale Azure + M365 Zero Trust 9 strati
- Migrazione identità: Samba4 AD + Google Identity → Entra ID Premium P2
- Migrazione posta: Zimbra (30 utenti) + Gmail (20 utenti) → Exchange Online
- Containerizzazione Odoo, Gitea, MediaWiki su AKS + PostgreSQL
- Migrazione Pondus: VM + Azure SQL MI con geo-replication
- Setup HPC: Azure HBv4 spot/on-demand, CycleCloud + Slurm, VPN dedicata
- Configurazione FortiGate 100F HA + VPN S2S + VLAN
- Microsoft Defender for Endpoint (50 device)
- Azure Backup + Site Recovery con test RPO/RTO
- Automazione IaC: Terraform, GitHub Actions, self-healing
- Penetration test full-scope
- Collaudo, go-live e dismissione on-premise
- Documentazione as-built e runbook operativi

---

## 10. Pacchetti Post-Go-Live (opzionali)

Tutti i pacchetti includono **6 mesi di formazione in sede (21 giornate)**: AZ-104 (2 sysadmin), SC-900 (team IT), affiancamento settimanale, M365 adoption workshop (50 utenti), security awareness + Attack Simulation.

| # | Pacchetto | Formazione | Presenza in sede | Assistenza | **Prezzo** |
|---|---|---|---|---|---|
| **1** | **Basic** | 6 mesi (21gg) | Solo formazione | Remoto base (ticket, SLA 24h) | **€18.000** |
| **2** | **Standard** | 6 mesi (21gg) | 2gg/mese (24gg/anno) | Remoto + in sede, SLA 8h | **€33.000** |
| **3** | **Plus** | 6 mesi (21gg) | 2gg/mese A1, 1gg/mese A2 | 2 anni copertura, SLA 8h | **€46.000** |
| **4** | **Premium** | 12 mesi (base+avanzata, 42gg) | 2gg/mese (48gg) | 24/7 reperibilità, SLA 4h | **€58.000** |

---

## 11. Budget Complessivo 2 Anni

### Scenario con pacchetto Standard (consigliato)

| Voce | Anno 1 | Anno 2 | Totale |
|---|---|---|---|
| Infrastruttura (pass-through) | €450.000 | €95.000\* | €545.000 |
| Costruzione infrastruttura (5Stack) | €120.000 | — | €120.000 |
| Pacchetto Standard (formazione + supporto) | €33.000 | — | €33.000 |
| **TOTALE** | **€603.000** | **€95.000\*** | **€698.000** |

\* HPC variabile escluso dai ricorrenti fissi: compute, storage ed egress a consumo (stimato €35.000–€125.000/anno)

### Confronto tra tutti i pacchetti

| | Basic | Standard | Plus | Premium |
|---|---|---|---|---|
| Infrastruttura Anno 1 | €450.000 | €450.000 | €450.000 | €450.000 |
| Costruzione 5Stack | €120.000 | €120.000 | €120.000 | €120.000 |
| Pacchetto post | €18.000 | €33.000 | €46.000 | €58.000 |
| **Totale Anno 1** | **€588.000** | **€603.000** | **€616.000** | **€628.000** |
| Infrastruttura Anno 2\* | €95.000 | €95.000 | €95.000 | €95.000 |
| **Totale 2 anni\*** | **€683.000** | **€698.000** | **€711.000** | **€723.000** |

\* HPC variabile escluso: compute, storage ed egress a consumo (stimato €35.000–€125.000/anno)

> Dal terzo anno: ~€95.000/anno di ricorrenti fissi infrastruttura (HPC variabile escluso). TP Group autonoma con pacchetto Basic/Standard/Plus.

---

## 12. Vantaggi della Soluzione

| Area | Vantaggio |
|---|---|
| **Riduzione complessità IT** | Da due stack eterogenei a unico ecosistema Microsoft + Azure |
| **Risparmio sui costi** | TCO ridotto del ~30% su 3 anni vs mantenimento on-prem originale |
| **Scalabilità** | Cloud elastico, nessun vincolo fisico (rack, raffreddamento, UPS) |
| **Sicurezza Zero Trust a 9 strati** | Difesa indipendente su ogni layer |
| **Business Continuity** | Nessun SPOF: FortiGate HA, Azure multi-region, geo-replication |
| **Audit e Compliance** | Log centralizzato, Purview per ISO 27001, eDiscovery |
| **HPC flessibile** | Spot per batch, on-demand per produzione, multi-region per DR |
| **Latenza zero in sede** | Azure File Sync con cache locale per file caldi |
| **Automazione** | IaC, self-healing, policy-as-code — riduce carico IT |
| **Visibilità** | Dashboard unificata Grafana + Azure Monitor + Sentinel |
| **Prevenzione proattiva** | Vulnerability scanning, threat hunting, Attack Simulation Training |
| **Efficienza team IT** | Da 8 a 6 risorse grazie ad automazione e dashboard |

---

## 13. Differenze v0.2 → v0.4

| Aspetto | v0.2 | v0.4 |
|---|---|---|---|
| **Budget infra anno 1** | €485.000 | **€450.000** (pass-through) |
| **Servizi 5Stack** | Incluso nel budget | Separato: €120.000 costruzione + €18k–€58k pacchetto post |
| **HPC compute** | 16 nodi bare-metal (€270k vincolati 3 anni) | Azure HBv4 (€230k anno 1, **a consumo** dal anno 2) |
| **HPC storage** | Incluso nel bare-metal | **€27.000** (Azure NetApp Files / Lustre) |
| **M365** | E5 (€33k) | E5 (€33k) |
| **Firewall** | FortiGate 100F HA (€15k) | FortiGate 100F HA — confermato |
| **Azure Firewall** | Standard (€10k) | Standard (€10k) — confermato |
| **Azure Support Plan** | Non presente | **€12.000/anno** (Standard, <1h risposta) |
| **Azure Bastion** | Non presente | **€2.000/anno** |
| **Microsoft Purview** | Non presente | **€5.000/anno** (compliance ISO 27001) |
| **Penetration test** | €12k | Incluso nella costruzione 5Stack |
| **Automation/IaC** | Non presente | Incluso nella costruzione 5Stack |
| **AKS** | Non presente | Incluso nella costruzione 5Stack |
| **Costi ricorrenti fissi** | ~€95.000 | **~€95.000** (+ HPC variabile a consumo) |
| **Team IT** | 8 pax | 6 pax (grazie ad automazione) |
| **Vendor lock-in HPC** | Alto (3 anni) | Basso (nessun vincolo) |

---

## 14. Rischi e Mitigazioni

| Rischio | Probabilità | Impatto | Mitigazione |
|---|---|---|---|
| Latenza HPC su Azure vs bare-metal | Media | Alto | Test workload in fase 5, assessment preliminare, eventuali nodi riservati |
| Complessità Kubernetes per team IT | Media | Medio | Formazione inclusa, template IaC pre-configurati |
| Costi Azure superiori al previsto | Media | Alto | Budget alert, RI, AHUB, spot instance, tag governance |
| Migrazione posta Zimbra/Gmail | Bassa | Alto | Fase 3 dedicata, cut-over pianificato, rollback plan |
| Resistenza al cambiamento utenti | Media | Basso | Formazione continua, attack simulation, adoption campaign |
| Dipendenza da 5Stack post go-live | Media | Medio | Knowledge transfer formale fase 9, IaC documentato |

---

*Documento da considerarsi come bozza preliminare aggiornata. Ogni sezione è suscettibile di modifica e approfondimento in base al confronto con il cliente.*

**TP Group srl - Proposta 5Stack v0.4**
