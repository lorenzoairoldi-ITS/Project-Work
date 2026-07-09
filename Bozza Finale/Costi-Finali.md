# Proposta TP Group — Costi Finali

**5Stack** — 09 Luglio 2026
**Team:** Lorenzo Airoldi (PM), Jacopo Merlo (Infra & HPC), Emanuele Sole (Net & Security), Gabriele Mattiolo (Cloud & App), Yassine Suriano (Doc & Integration)

---

## Anno 1 — Infrastruttura (pass-through)

| Voce | Costo |
|---|---|
| **Microsoft 365 E5** (50 utenti) | €33.000 |
| **Azure IaaS/PaaS** (VM, SQL MI, AKS, PostgreSQL) | €40.000 |
| **Azure Firewall Standard** + NSG + DDoS Protection | €10.000 |
| **Azure Backup + Site Recovery** (DR North Europe) | €8.000 |
| **HPC — Azure HBv4** spot + on-demand (16 nodi elastici) | €230.000 |
| *Breakdown HPC:* | |
| *Compute on-demand (30%): 16 x 1.500h x ~€4,80/h* | *€115.200* |
| *Compute spot (70%): 16 x 3.500h x ~€1,45/h* | *€81.200* |
| *Storage (NetApp Files 50TB premium)* | *€27.000* |
| *Egress (50TB/anno)* | *€10.000* |
| *Setup iniziale + benchmark* | *€26.600* |
| **HPC Storage** — Azure NetApp Files / Managed Lustre | €27.000 |
| **HPC Egress** — dati in uscita verso sede | €10.000 |
| **Azure Support Plan Standard** (risposta <1h) | €12.000 |
| **Azure Bastion** (accesso sicuro VM) | €2.000 |
| **Microsoft Purview** (compliance, ISO 27001) | €5.000 |
| **FortiGate 100F HA pair** (copre 3 anni) | €15.000 |
| **Microsoft Sentinel** (SIEM, opzionale) | €5.000 |
| **Subtotale** | **€397.000** |
| Fondo imprevisti (~13%) | **€53.000** |
| **TOTALE INFRASTRUTTURA** | **€450.000** |

## Anno 2+ — Ricorrenti fissi

| Voce | Costo/anno |
|---|---|
| Microsoft 365 E5 | €33.000 |
| Azure IaaS/PaaS (produzione) | €30.000 |
| Azure Backup + Site Recovery | €6.000 |
| Azure Support Plan Standard | €12.000 |
| Azure Bastion | €2.000 |
| Microsoft Purview | €5.000 |
| Microsoft Sentinel | €5.000 |
| Licenze varie | €2.000 |
| **TOTALE** | **~€95.000/anno** |

> **HPC a consumo**: compute, storage ed egress sono variabili in base all'uso effettivo — stimato €35.000–€125.000/anno

---

## Servizi 5Stack — Costruzione (una tantum)

### €120.000 — forfait 12 settimane

| Ruolo | Tariffa | Giorni | Importo |
|---|---|---|---|
| Project Manager | €650/gg | 26gg | €16.900 |
| Sistemisti Senior (x4) | €500/gg | 161gg | €80.500 |
| Trasferta | — | — | €1.100 |
| **Costo effettivo** | | | **€98.500** |
| **Forfait 5Stack** | | | **€120.000** |
| *Margine* | | | *18%* |

### Cosa include:
- Progettazione architetturale Azure + M365 + Zero Trust
- Migrazione identita, posta, file, applicazioni
- Setup HPC su Azure HBv4 con CycleCloud + Slurm
- Configurazione FortiGate 100F HA + VPN S2S + VLAN
- Microsoft Defender for Endpoint (50 device)
- Azure Backup + Site Recovery con test RPO/RTO
- Automazione IaC (Terraform, GitHub Actions, self-healing)
- Penetration test full-scope
- Documentazione as-built e runbook operativi

---

## Pacchetti Post-Go-Live (opzionali)

Tutti includono **6 mesi di formazione in sede (21 giornate)**: AZ-104, SC-900, affiancamento settimanale, M365 adoption, security awareness.

| # | Pacchetto | Formazione | Presenza | Assistenza | **Prezzo** |
|---|---|---|---|---|---|
| 1 | **Basic** | 6 mesi (21gg) | Solo formazione | Remoto, SLA 24h | **€18.000** |
| 2 | **Standard** | 6 mesi (21gg) | 2gg/mese (24gg/anno) | Remoto + in sede, SLA 8h | **€33.000** |
| 3 | **Plus** | 6 mesi (21gg) | 2gg/mese anno 1, 1gg/mese anno 2 | 2 anni copertura, SLA 8h | **€46.000** |
| 4 | **Premium** | 12 mesi (42gg) | 2gg/mese (48gg) | 24/7, SLA 4h | **€58.000** |

---

## Scenari Completi

### Base (nessun pacchetto post)

| Anno | Importo |
|---|---|
| Anno 1 — Infrastruttura €450k + Costruzione €120k | **€570.000** |
| Anno 2 | **€95.000** (+ HPC a consumo) |
| Anno 3+ | **€95.000/anno** (+ HPC a consumo) |

### Standard (consigliato)

| Anno | Importo |
|---|---|
| Anno 1 — €450k + €120k + €33k | **€603.000** |
| Anno 2 | **€95.000** (+ HPC a consumo) |
| Anno 3+ | **€95.000/anno** (+ HPC a consumo) |

### Confronto completo

| | Base | Basic | Standard | Plus | Premium |
|---|---|---|---|---|---|
| **Anno 1** | €570.000 | €588.000 | €603.000 | €616.000 | €628.000 |
| **Anno 2*** | €95.000 | €95.000 | €95.000 | €95.000 | €95.000 |
| **2 anni*** | €665.000 | €683.000 | €698.000 | €711.000 | €723.000 |

* HPC variabile escluso (35k-125k/anno a consumo)

---

## TCO 3 Anni — Confronto

| Voce | Full Cloud (proposta) | On-premise potenziato |
|---|---|---|
| **Anno 1** | €603.000 | €792.000* |
| **Anno 2** | €95.000 | €297.000** |
| **Anno 3** | €95.000 | €297.000** |
| **Totale 3 anni (fisso)** | **€793.000** | **€1.386.000** |
| **HPC variabile (3 anni)** | ~€150.000 | Incluso |
| **TOTALE 3 ANNI** | **~€943.000** | **~€1.386.000** |
| **Risparmio** | | **~€443.000 (~32%)** |

> *On-prem Anno 1: rack + raffreddamento €50k, UPS €20k, 3 server €90k, SAN 50TB €60k, 8 nodi HPC €270k, FortiGate HA €15k, backup €30k, 2 sistemisti €120k, VMware €15k, M365 E3 €22k, consulenza €50k, imprevisti €50k
> **On-prem Anno 2-3: 2 sistemisti €120k, manutenzione HW €25k, elettricità/raffreddamento €30k, ammortamento HPC €90k, M365 E3 €22k, VMware €10k

---

## Riepilogo Rapido

| Cosa | Importo |
|---|---|
| Infrastruttura Anno 1 (pass-through) | €450.000 |
| Costruzione 5Stack (una tantum) | €120.000 |
| Ricorrenti fissi da Anno 2 | ~€95.000/anno |
| HPC (variabile) | €35k-125k/anno |
| Pacchetti post (opzionali) | €18k-58k |
| **Investimento totale primo anno** | **€570.000 - €628.000** |
