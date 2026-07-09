# Riepilogo Esecutivo — TP Group srl

**Proposta di Integrazione IT | 5Stack | 09 Luglio 2026**

---

## Il Progetto in 3 Righe

Fusione di **Thema Consulting** (30 pax, on-prem) e **Pro Studio** (20 pax, cloud) in **TP Group srl** (50 pax). Passaggio a **Full Cloud + Zero Trust** con Microsoft Azure e M365.

---

## Architettura Target

| Layer | Soluzione |
|---|---|
| **Identity** | Microsoft Entra ID Premium P2 |
| **Collaborazione** | Microsoft 365 E5 (Exchange, Teams, SharePoint, OneDrive) |
| **Applicazioni** | Azure VM + AKS + SQL Managed Instance |
| **HPC** | Azure HBv4 spot/on-demand (16 nodi, CycleCloud + Slurm) |
| **Sicurezza** | Zero Trust a 9 strati: MFA, Defender, DLP, Azure Firewall, FortiGate 100F HA |
| **Backup & DR** | Azure Backup + Site Recovery + SQL MI geo-replication (RPO 15min) |

---

## Roadmap: 12 Settimane

1. **Preparazione** — Assessment, contratti (Sett. 1-2)
2. **Identity** — Entra ID, MFA, Conditional Access (Sett. 2-3)
3. **Collaborazione** — Migrazione posta, file su M365 (Sett. 3-5)
4. **Applicazioni** — Pondus su SQL MI, Odoo su AKS (Sett. 4-8)
5. **HPC** — Azure HBv4, CycleCloud (Sett. 3-6)
6. **Sicurezza & DR** — FortiGate HA, Defender, Backup (Sett. 5-8)
7. **Governance** — IaC, Policy, RBAC (Sett. 8-9)
8. **Collaudo** — Pen test, test DR, formazione (Sett. 9-11)
9. **Chiusura** — Dismissione on-prem, handover (Sett. 11-12)

---

## Investimento

| Voce | Importo |
|---|---|
| Infrastruttura Anno 1 (pass-through) | **€450.000** |
| Costruzione 5Stack (una tantum) | **€120.000** |
| Ricorrenti fissi da Anno 2 | **~€95.000/anno** |
| HPC (variabile a consumo) | €35k-125k/anno |
| Pacchetti assistenza (opzionali) | €18k-58k |
| **Investimento totale Anno 1** | **€570.000 - €628.000** |

---

## Vantaggi Chiave

- **Zero vincoli fisici** — niente rack saturi, condizionamento, UPS
- **Sicurezza ISO 27001** — Zero Trust, MFA, DLP, Audit, DR multi-region
- **HPC flessibile** — spot per batch, on-demand per produzione, nessun lock-in
- **Automazione** — IaC (Terraform), self-healing, dashboard unificata
- **Riduzione TCO** ~30% su 3 anni rispetto a mantenimento on-prem

---

*Documento di sintesi — Proposta completa in Proposta-TP-Group-v1.0.md*
