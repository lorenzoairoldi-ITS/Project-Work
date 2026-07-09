# Proposta TP Group — Costi v0.4

**5Stack** — 06 Luglio 2026

---

## In sintesi

| Cosa | Anno 1 | Anno 2 in poi |
|---|---|---|
| Infrastructure (Azure, M365, HW) | **€450.000** | **~€95.000/anno** (+ HPC se lo usi) |
| Costruzione 5Stack (una tantum) | **€120.000** | — |
| Assistenza post (opzionale) | **€18.000–€58.000** | — |

---

## 1. Cosa paghi a Microsoft e fornitori (Year 1)

| Voce | Costo |
|---|---|
| Microsoft 365 E5 (50 utenti) | €33.000 |
| Azure macchine e servizi (VM, SQL, AKS…) | €40.000 |
| Azure Firewall Standard (con filtro TLS) | €10.000 |
| Backup e Disaster Recovery | €8.000 |
| HPC calcolo — Azure HBv4 (16 nodi) | €230.000 |
| HPC storage — Azure NetApp / Lustre | €27.000 |
| HPC dati in uscita verso sede | €10.000 |
| Supporto tecnico Microsoft (risposta <1h) | €12.000 |
| Azure Bastion (accesso sicuro server) | €2.000 |
| Microsoft Purview (compliance dati) | €5.000 |
| FortiGate 100F HA (firewall sede, 3 anni) | €15.000 |
| Microsoft Sentinel (sicurezza, opzionale) | €5.000 |
| **Subtotale** | **€397.000** |
| Fondo imprevisti (~13%) | **€53.000** |
| **TOTALE** | **€450.000** |

## 2. Cosa paghi a Microsoft e fornitori (Year 2+)

Ogni anno, finché tieni l'infrastruttura.

| Voce | Costo/anno |
|---|---|
| Microsoft 365 E5 | €33.000 |
| Azure macchine e servizi | €30.000 |
| Backup e Disaster Recovery | €6.000 |
| Supporto tecnico Microsoft | €12.000 |
| Azure Bastion | €2.000 |
| Microsoft Purview | €5.000 |
| Microsoft Sentinel | €5.000 |
| Licenze varie | €2.000 |
| **TOTALE** | **€95.000/anno** |

> **HPC a parte**: lo paghi solo quando lo usi. Calcolo €15k–80k/anno + storage €15k–30k/anno + dati in uscita €5k–15k/anno.

---

## 3. Cosa paghi a 5Stack per costruire tutto

**€120.000 — una tantum.** Per 12 settimane di lavoro, team dedicato:

| Ruolo | Tariffa |
|---|---|
| 1 Project Manager | €650/giorno |
| 4 Sistemisti senior | €500/giorno |

### Cosa include

- Progettazione Azure + Microsoft 365
- Migrazione utenti, posta, file, app
- Configurazione HPC su Azure
- FortiGate, sicurezza, backup, disaster recovery
- Automazione (Terraform, GitHub Actions)
- Penetration test, collaudo, go-live
- Documentazione e formazione base
- Dismissione server vecchi

### Perché €120.000

| KPI | Valore |
|---|---|---|
| Giorni-uomo totali (187gg = 26gg PM + 161gg sistemisti) | 5 persone in parallelo su 12 settimane |
| Costo effettivo (giorni × tariffe) | ~€98.500 |
| Margine per 5Stack | ~€21.500 (18%) |
| Impegno del team | 3 mesi al 60% |

---

## 4. Pacchetti Assistenza (opzionali — dopo la formazione)

La **formazione 6 mesi in sede (€15k)** è già inclusa nel forfait e copre: AZ-104, SC-900, affiancamento settimanale, workshop M365, security awareness.

I pacchetti qui sotto sono **solo supporto tecnico**, si attivano dopo la formazione e durano 1 anno.

---

### Light

| Cosa include | Dettaglio |
|---|---|
| Assistenza remota | Ticket + email + telefono, orario ufficio |
| SLA | Risposta entro **24h lavorative** |
| Presenza in sede | Nessuna |
| Report | Report trimestrale stato infrastruttura |
| **Prezzo** | **€5.000** |

> Per chi ha un team IT interno capace e vuole solo un backup remoto.

---

### On-Site

| Cosa include | Dettaglio |
|---|---|
| Assistenza remota | Ticket + email + telefono, orario ufficio |
| SLA | Risposta entro **8h lavorative** |
| Presenza in sede | **2 giorni al mese** (24gg/anno) per affiancamento operativo |
| Report | Report mensile stato infrastruttura |
| **Prezzo** | **€20.000** |

> Per chi vuole un tecnico 5Stack presente periodicamente per verifiche, aggiornamenti e supporto diretto.

---

### Shield

| Cosa include | Dettaglio |
|---|---|
| Assistenza remota | Ticket + email + telefono, **24 ore su 24** |
| SLA | Risposta entro **4h**, anche notte e weekend |
| Presenza in sede | **2 giorni al mese** (24gg/anno) per interventi programmati |
| Reperibilità | Tecnico di turno h24 con escalation garantita |
| Extra | **Penetration test annuale** incluso + report mensile |
| **Prezzo** | **€35.000** |

> Per chi non ha figure IT interne e vuole dormire sonni tranquilli.

---

## 5. Totale complessivo — scegli il tuo scenario

La base comune per tutti è:
**€585.000** = €450.000 (infrastruttura) + €120.000 (costruzione) + €15.000 (formazione 6 mesi)

Aggiungi il pacchetto assistenza che preferisci:

### Solo Formazione (nessun pacchetto)

| Anno | Cosa paghi | Totale |
|---|---|---|
| **Anno 1** | Base €585k | **€585.000** |
| **Anno 2** | Infrastruttura ricorrente €95k (+ HPC se lo usi) | **€95.000** |
| **Anno 3+** | Infrastruttura ricorrente €95k (+ HPC se lo usi) | **€95.000/anno** |

### + Light (+€5k)

| Anno | Cosa paghi | Totale |
|---|---|---|
| **Anno 1** | Base €585k + Light €5k | **€590.000** |
| **Anno 2** | Infrastruttura ricorrente €95k (+ HPC se lo usi) | **€95.000** |
| **Anno 3+** | Infrastruttura ricorrente €95k (+ HPC se lo usi) | **€95.000/anno** |

### + On-Site (+€20k) — consigliato

| Anno | Cosa paghi | Totale |
|---|---|---|
| **Anno 1** | Base €585k + On-Site €20k | **€605.000** |
| **Anno 2** | Infrastruttura ricorrente €95k (+ HPC se lo usi) | **€95.000** |
| **Anno 3+** | Infrastruttura ricorrente €95k (+ HPC se lo usi) | **€95.000/anno** |

### + Shield (+€35k)

| Anno | Cosa paghi | Totale |
|---|---|---|
| **Anno 1** | Base €585k + Shield €35k | **€620.000** |
| **Anno 2** | Infrastruttura ricorrente €95k (+ HPC se lo usi) | **€95.000** |
| **Anno 3+** | Infrastruttura ricorrente €95k (+ HPC se lo usi) | **€95.000/anno** |

---

## Riepilogo rapido

| | Base | + Light | + On-Site | + Shield |
|---|---|---|---|---|
| Anno 1 | €585k | €590k | €605k | €620k |
| Anno 2 | €95k | €95k | €95k | €95k |
| Anno 3+ | €95k/anno | €95k/anno | €95k/anno | €95k/anno |

> **HPC a consumo**: si aggiunge a questi importi solo quando usi il calcolo (stimato €35k–125k/anno in base all'utilizzo)
