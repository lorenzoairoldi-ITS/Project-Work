# RIEPILOGO COSTI — Proposta TP Group

---

## 1. COSA PAGA TP GROUP A MICROSOFT/VENDITORI (pass-through)

| Voce | Anno 1 | Anno 2 | Note |
|---|---|---|---|---|
| M365 E5 (50 utenti) | €33.000 | €33.000 | Upgrade da BP+E3 — include Purview, Defender P2, Compliance |
| Azure IaaS/PaaS (VM, SQL MI, AKS...) | €40.000 | €30.000 | Con RI + AHUB |
| Azure Firewall Standard | €10.000 | — | |
| Azure Backup + Site Recovery | €8.000 | €6.000 | |
| HPC HBv4 (16 nodi spot + on-demand) | €230.000 | a consumo | Stimato €15k–€80k/anno in base all'uso |
| HPC storage (NetApp Files / Lustre) | €27.000 | a consumo | Storage ad alte prestazioni per carichi HPC |
| HPC egress (dati in uscita) | €10.000 | a consumo | Trasferimento dati HPC verso sede |
| Azure Support Plan Standard | €12.000 | €12.000 | Risposta <1h, indispensabile per HPC in produzione |
| Azure Bastion | €2.000 | €2.000 | Accesso sicuro VM senza IP pubblico |
| Microsoft Purview | €5.000 | €5.000 | Data governance, compliance ISO 27001 |
| FortiGate 100F HA (copre 3 anni) | €15.000 | — | |
| Microsoft Sentinel (opzionale) | €5.000 | €5.000 | |
| Licenze varie | — | €2.000 | |
| **Subtotale** | **€397.000** | **€95.000\*** | \* escluso HPC variabile (compute+storage+egress) |
| Buffer imprevisti (~13%) | **€53.000** | — | |
| **TOTALE INFRASTRUTTURA** | **€450.000** | **~€95.000/anno\*** | |

> \* HPC escluso dai ricorrenti fissi: compute, storage ed egress sono a consumo in base all'uso effettivo (stimato complessivo €35.000–€125.000/anno)

---

## 2. COSA PAGA TP GROUP A 5STACK PER COSTRUIRE

### Tariffe giornaliere 5Stack

| Ruolo | €/giorno | Nr. |
|---|---|---|---|
| Project Manager | €650 | 1 |
| Sistemista Senior | €500 | 4 |

### Calcolo giorni × tariffa per ogni fase

| Fase | PM (€650) | Sistemisti (€500) | **Conto** |
|---|---|---|---|---|
| 1. Preparazione | 3gg × 650 = €1.950 | 5gg × 500 = €2.500 | **€4.450** |
| 2. Identity | 1gg × 650 = €650 | 9gg × 500 = €4.500 | **€5.150** |
| 3. Collaborazione | 2gg × 650 = €1.300 | 18gg × 500 = €9.000 | **€10.300** |
| 4. Applicazioni | 5gg × 650 = €3.250 | 35gg × 500 = €17.500 | **€20.750** |
| 5. HPC | 2gg × 650 = €1.300 | 18gg × 500 = €9.000 | **€10.300** |
| 6. Sicurezza & DR | 3gg × 650 = €1.950 | 27gg × 500 = €13.500 | **€15.450** |
| 7. Governance | 2gg × 650 = €1.300 | 14gg × 500 = €7.000 | **€8.300** |
| 8. Collaudo | 5gg × 650 = €3.250 | 23gg × 500 = €11.500 | **€14.750** |
| 9. Chiusura | 3gg × 650 = €1.950 | 12gg × 500 = €6.000 | **€7.950** |
| **Subtotale** | **26gg = €16.900** | **161gg = €80.500** | **€97.400** |
| Trasferta | | | **€1.100** |
| **TOTALE** | | | **€120.000** |

> **Nota:** €120k non è la somma dei giorni (che è ~€100k). È un forfait maggiorato perché 5Stack blocca il 60% del team per 3 mesi su un progetto specialistico (HPC + AKS + Zero Trust).

---

## 3. COSA PAGA TP GROUP A 5STACK DOPO (opzionale)

Tutti i pacchetti includono **6 mesi di formazione in sede (21gg)**

| # | Pacchetto | Include | Prezzo |
|---|---|---|---|
| **1** | **Basic** | Formazione 6 mesi + supporto remoto base (SLA 24h) | **€18.000** |
| **2** | **Standard** | Formazione 6 mesi + 2gg/mese in sede per 1 anno (SLA 8h) | **€33.000** |
| **3** | **Plus** | Formazione 6 mesi + 2gg/mese A1 + 1gg/mese A2 (2 anni, SLA 8h) | **€46.000** |
| **4** | **Premium** | Formazione 6 mesi + 6 mesi avanzata + 2gg/mese + 24/7 (SLA 4h) | **€58.000** |

---

## 4. TOTALE PER IL CLIENTE (esempi)

| | Basic | Standard | Plus | Premium |
|---|---|---|---|---|---|
| Infrastruttura | €450.000 | €450.000 | €450.000 | €450.000 |
| Costruzione 5Stack | €120.000 | €120.000 | €120.000 | €120.000 |
| Pacchetto post | €18.000 | €33.000 | €46.000 | €58.000 |
| **Anno 1** | **€588.000** | **€603.000** | **€616.000** | **€628.000** |
| **Anno 2\*** | **€95.000** | **€95.000** | **€95.000** | **€95.000** |
| **2 anni\*** | **€683.000** | **€698.000** | **€711.000** | **€723.000** |

> \* HPC escluso dai ricorrenti fissi: compute, storage ed egress sono a consumo (stimato complessivo €35.000–€125.000/anno)

---

## 5. PERCHÉ €120k PER 5STACK

| KPI | Valore |
|---|---|---|
| Giorni-uomo totali | 187gg (26gg PM + 161gg sistemisti) |
| Costo effettivo (tariffe × giorni) | ~€98.500 |
| **Forfait costruzione** | **€120k** |
| Margine sul forfait | ~€21.500 (18%) |
| Giorni in cui il team è impegnato sul progetto | ~15gg/settimana su 25 disponibili = 60% per 3 mesi |
| Composizione team | 1 PM (€650/gg) + 4 sistemisti (€500/gg) |
