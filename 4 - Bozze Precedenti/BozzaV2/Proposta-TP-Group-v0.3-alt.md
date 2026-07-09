# Proposta di Integrazione IT — TP Group srl v0.3

**5Stack** — 06 Luglio 2026
**Budget primo anno:** ~€450.000
**Costi ricorrenti:** ~€85.000/anno
**Budget complessivo 2 anni:** ~€601.000

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

## 8. Budget — Primo Anno (~€450k)

| Voce | Costo | Note |
|---|---|---|
| **Infrastruttura Core** | | |
| M365 Business Premium + E3 Security (50 utenti) | €23.000 | |
| Azure IaaS/PaaS (VM, SQL MI, AKS, PostgreSQL) | €40.000 | Con RI + Azure Hybrid Benefit |
| Azure Firewall Basic + NSG + DDoS Protection | €6.000 | |
| Azure Backup + Site Recovery | €8.000 | |
| **HPC** | | |
| Azure HBv4 spot + on-demand (16 nodi elastici) | €200.000 | |
| **Rete Sede** | | |
| FortiGate 100F HA pair (3 anni) | €15.000 | |
| **Automation & IaC** | | |
| Terraform + GitHub Actions + self-healing | €15.000 | |
| **Efficienza Operativa** | | |
| Azure File Sync + cache locale sede | €5.000 | |
| Grafana dashboard + Azure Monitor custom | €5.000 | |
| AKS setup + containerizzazione Odoo/Gitea/Wiki | €15.000 | |
| Bastion host + JIT RDP/SSH | €2.000 | |
| Assessment HPC workload | €5.000 | |
| **Servizi Professionali** | | |
| Servizi 5Stack (migrazione + setup + formazione) | €45.000 | |
| Penetration test (full-scope, fase 8) | €7.000 | |
| Formazione + Attack Simulation Training | €8.000 | |
| **SIEM** | | |
| Microsoft Sentinel (opzionale, da mese 6) | €5.000 | |
| **Subtotale** | **€404.000** | |
| **Buffer imprevisti** | **~€46.000** | |
| **TOTALE PRIMO ANNO** | **~€450.000** | |

### Costi Ricorrenti ~€85.000/anno

| Voce | Costo/anno |
|---|---|
| M365 Business Premium + E3 Security | €23.000 |
| Azure IaaS/PaaS (produzione) | €30.000 |
| Azure Backup + Site Recovery | €6.000 |
| HPC (on-demand + spot) | €15.000 |
| Sentinel (se attivato) | €5.000 |
| FortiGate 100F (già coperto 3 anni) | — |
| Formazione continua | €4.000 |
| Licenze varie | €2.000 |
| **Totale** | **~€85.000** |

---

## 9. Servizi 5Stack — Formazione & Supporto (2 anni)

Piano per rendere TP Group pienamente autonomo in 24 mesi.

### Anno 1 — Fondazione & Affiancamento

| Voce | Dettaglio | Costo |
|---|---|---|
| **Formazione IT** | AZ-104 (2 sysadmin) + SC-900 (tutto team IT). Affiancamento on-site 1gg/settimana (mesi 1-4) | €13.000 |
| **Formazione utenti** | M365 adoption workshop (50 dipendenti), security awareness base, Attack Simulation | €8.000 |
| **Managed Support** | Monitoring 24/7, help desk reactive, patching mensile, audit trimestrale, SLA 4h, supporto HPC | €15.000 |
| **Documentazione** | As-built, runbook operativi, procedure DR testate | incluso |
| **Totale Anno 1** | | **€36.000** |

### Anno 2 — Autonomia Guidata

| Voce | Dettaglio | Costo |
|---|---|---|
| **Formazione IT avanzata** | AZ-305 (2 sysadmin) + SC-200 (1 security engineer) + workshop IaC/Terraform | €15.000 |
| **Formazione utenti** | Refresher sicurezza, phishing campaign avanzata | €3.000 |
| **Managed Support** | On-call escalation, supporto HPC su richiesta, SLA 8h | €5.000 |
| **Penetration test** | Full-scope pre-handover | €7.000 |
| **Handover finale** | Certificazione autonomia, test DR supervisionato, documentazione finale | incluso |
| **Totale Anno 2** | | **€30.000** |

### Riepilogo 2 anni

| | Anno 1 | Anno 2 | Totale |
|---|---|---|---|
| Formazione IT | €13.000 | €15.000 | €28.000 |
| Formazione utenti | €8.000 | €3.000 | €11.000 |
| Managed Support | €15.000 | €5.000 | €20.000 |
| Penetration test | — | €7.000 | €7.000 |
| **Totale Servizi** | **€36.000** | **€30.000** | **~€66.000** |

### Plus aggiuntivo — Certificazioni Microsoft (opzionale)

*Attivabile su richiesta del cliente. Prezzi indicativi soggetti a variazioni listini Microsoft.*

| Certificazione | Ruolo | Costo |
|---|---|---|
| AZ-104 (Azure Administrator) | 2x Sysadmin | ~€3.300 |
| AZ-305 (Azure Solutions Architect) | 2x Sysadmin | ~€3.300 |
| SC-200 (Security Operations Analyst) | 1x Security Engineer | ~€1.700 |
| SC-900 (Security Fundamentals) | 3x Help Desk / altri | ~€2.100 |
| **Totale** | **6 persone** | **~€10.400** |

---

## 10. Budget Complessivo 2 Anni

| | Anno 1 | Anno 2 | Totale |
|---|---|---|---|
| Infrastruttura | €450.000 | €85.000 | €535.000 |
| Servizi 5Stack | €36.000 | €30.000 | €66.000 |
| **TOTALE** | **€486.000** | **€115.000** | **~€601.000** |

> Dal terzo anno: solo ~€85.000/anno di ricorrenti infrastruttura (TP Group autonoma)

---

## 11. Vantaggi della Soluzione

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

## 12. Differenze v0.2 → v0.3

| Aspetto | v0.2 | v0.3 |
|---|---|---|
| **Budget** | €485.000 | €450.000 |
| **HPC** | 16 nodi bare-metal OVH/Hetzner (€270k vincolati 3 anni) | Azure HBv4 spot + on-demand (€200k elastici) |
| **M365** | E5 (€33k) | Business Premium + E3 Security (€23k) |
| **Firewall** | FortiGate 100F HA (€15k) | FortiGate 100F HA — confermato |
| **Azure Firewall** | Standard (€10k) | Basic (€6k) |
| **Penetration test** | €12k | €7k |
| **Automation/IaC** | Non presente | Terraform + GitHub Actions + self-healing (€15k) |
| **AKS** | Non presente | Containerizzazione Odoo/Gitea/Wiki (€15k) |
| **Azure File Sync** | Non presente | Cache locale sede (€5k) |
| **Dashboard** | Non presente | Grafana + Azure Monitor (€5k) |
| **Assessment HPC** | Non presente | Ottimizzazione workload (€5k) |
| **Costi ricorrenti** | ~€95.000 | ~€85.000 |
| **Team IT** | 8 pax | 6 pax (grazie ad automazione) |
| **Vendor lock-in HPC** | Alto (3 anni) | Basso (nessun vincolo) |

---

## 13. Rischi e Mitigazioni

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

**TP Group srl - Proposta 5Stack v0.3**
