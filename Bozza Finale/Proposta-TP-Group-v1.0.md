# Proposta di Integrazione IT — TP Group srl v1.0

**5Stack** — 09 Luglio 2026
**Team:** Lorenzo Airoldi (PM), Jacopo Merlo (Infra & HPC), Emanuele Sole (Net & Security), Gabriele Mattiolo (Cloud & App), Yassine Suriano (Doc & Integration)

**Infrastruttura Anno 1:** €450.000 (pass-through)
**Costruzione 5Stack:** €120.000 (forfait)
**Ricorrenti fissi da Anno 2:** ~€95.000/anno (+ HPC variabile a consumo)

---

## 1. Pagina Introduttiva

| Dato | Valore |
|---|---|
| **Cliente** | TP Group srl (fusa da Thema Consulting + Pro Studio) |
| **Oggetto** | Offerta tecnico-economica per convergenza e potenziamento infrastrutture IT |
| **Data** | 09 Luglio 2026 |
| **Proponente** | 5Stack |
| **Durata progetto** | 12 settimane |
| **Modello architetturale** | Full Cloud + Zero Trust |

La presente proposta descrive la strategia, l'architettura target e il piano economico per unificare le infrastrutture IT di Thema Consulting (~30 dipendenti, on-premise) e Pro Studio (~20 dipendenti, cloud-native) nella nuova entita **TP Group srl** (50 dipendenti, sede unica a Torino).

---

## 2. Indice

1. Pagina Introduttiva
2. Indice
3. Presentazione del Team (5Stack)
4. Introduzione — Sintesi dell'Offerta
5. Assunzioni — Prerequisiti per l'Erogazione
6. Descrizione Fornitura Hardware
7. Attivita Richieste — Installazione, Test, Migrazione
8. Documentazione Rilasciata
9. Termini e Condizioni
10. Quotazione Economica

**Appendici:**
- A. Architettura Zero Trust — Dettaglio 9 Strati
- B. Roadmap di Progetto (Gantt)
- C. Tabella RASCI

---

## 3. Presentazione del Team (5Stack)

### Chi Siamo

**5Stack** e un team di consulenza IT specializzato in infrastrutture cloud, cybersecurity e trasformazione digitale per PMI. Operiamo su progetti di migrazione al cloud, consolidamento data center e adeguamento normativo (ISO 27001).

### Il Team di Progetto

| Ruolo | Nome | Competenze |
|---|---|---|
| **Project Manager** | Lorenzo Airoldi | Gestione progetti IT, pianificazione, coordinamento team, interfaccia cliente |
| **Sistemista Senior — Infra & HPC** | Jacopo Merlo | Virtualizzazione (VMware, Hyper-V), HPC, InfiniBand, storage, backup |
| **Network & Security Engineer** | Emanuele Sole | Firewall (FortiGate), VPN, sicurezza perimetrale, Zero Trust, ISO 27001 |
| **Cloud & Application Engineer** | Gabriele Mattiolo | Azure IaaS/PaaS, SQL MI, AKS, Pondus, Odoo, automazione IaC |
| **IT Integration & Documentation Specialist** | Yassine Suriano | Migrazione servizi, documentazione tecnica, formazione utenti |

### Servizi Offerti

- Progettazione architetturale cloud (Azure, M365)
- Migrazione infrastrutture on-premise al cloud
- Sicurezza informatica (Zero Trust, ISO 27001)
- Automazione infrastruttura (Terraform, CI/CD)
- Formazione e affiancamento team IT interno

### Storia

5Stack e stato costituito per rispondere a progetti complessi di trasformazione digitale, con un focus specifico su PMI italiane del settore manifatturiero e automotive. Il team vanta esperienze pregresse in progetti di migrazione cloud, consolidamento data center e adeguamento a normative di settore.

---

## 4. Introduzione — Sintesi dell'Offerta

5Stack propone per TP Group srl un'architettura **Full Cloud + Zero Trust a 9 strati**, che risolve tutte le criticita emerse dall'analisi AS-IS:

### Problemi Risolti

| Criticita | Soluzione |
|---|---|
| Rack saturi, condizionamento al limite, UPS 10 min | Zero hardware on-premise — tutto su Azure e SaaS |
| FortiGate FG-60F sottodimensionato, no HA | FortiGate 100F HA pair + Azure Firewall |
| Backup solo on-premise (QNAP) | Azure Backup + Site Recovery multi-region |
| Nessuna protezione endpoint | Microsoft Defender for Endpoint (tutti i device) |
| Nessun MFA, audit, DLP | Entra ID P2 + Conditional Access + Purview |
| Samba4 AD limitato | Migrazione a Entra ID Premium P2 |
| HPC vincolato a on-prem (8 nodi) | 16 nodi Azure HBv4 elastici (spot + on-demand) |
| Pondus su HW fisico non ridondato | VM IIS + Azure SQL MI con geo-replication |
| Due stack eterogenei (Thema on-prem / Pro Studio cloud) | Ecosistema unico Microsoft + Azure |

### Principi Guida

- **Full Cloud first** — nessuna nuova infrastruttura on-premise
- **Zero Trust** — nessuna fiducia implicita, verifica ogni richiesta su 9 strati indipendenti
- **IaC (Infrastructure as Code)** — tutto l'ambiente definito come codice (Terraform)
- **Security by design** — sicurezza integrata in ogni layer, non aggiunta a posteriori

---

## 5. Assunzioni — Prerequisiti per l'Erogazione

### Assunzioni Generali

1. TP Group dispone di un contratto Microsoft Enterprise Agreement o CSP attivabile per M365/Azure
2. La connettivita Internet nella sede di Torino e adeguata (almeno 500 Mbps simmetrici, preferibilmente 1 Gbps)
3. I domini email di Thema e Pro Studio sono gestiti da TP Group per la configurazione MX/DKIM/DMARC
4. I dipendenti di Pro Studio hanno laptop aziendali compatibili con Windows 11 e Microsoft 365
5. Il commercialista di TP Group e coinvolto attivamente nella migrazione contabile (Profis -> Odoo)
6. Il periodo di parallelo contabile (1 mese fiscale) e accettato dalla direzione
7. Le chiavi di crittografia e i segreti aziendali sono resi disponibili a 5Stack per la configurazione iniziale
8. TP Group nomina un referente interno per il progetto (IT manager o persona delegata)

### Esclusioni

- Licenze software pre-esistenti (es. eventuali licenze VMware, Windows Server) non incluse nel perimetro
- Migrazione di dati da sistemi non elencati nell'analisi AS-IS
- Sviluppo software o modifica del codice sorgente di Pondus
- Supporto a dispositivi non Windows (es. tablet, smartphone di proprieta del dipendente)
- Formazione su prodotti non inclusi nel pacchetto post-go-live scelto
- Servizi di gestione IT continuativa oltre il pacchetto post-go-live sottoscritto

---

## 6. Descrizione Fornitura Hardware

### 6.1 Apparecchiature

| Componente | Qta | Specifiche | Costo (copre 3 anni) |
|---|---|---|---|
| FortiGate 100F HA pair | 2 | NGFW, 10 GbE, IPS/IDS, SSL inspection, VPN Throughput 1 Gbps | €15.000 |

### 6.2 Infrastruttura Cloud (pass-through)

| Risorsa | Specifiche | Costo Annuo |
|---|---|---|
| **Azure HBv4** — 16 nodi elastici | Spot per batch, on-demand per produzione. CycleCloud + Slurm | €230.000 (Anno 1) — vedi breakdown sotto |
| **Azure NetApp Files / Managed Lustre** | Storage HPC ad alte prestazioni | €27.000 (Anno 1) |
| **SQL Managed Instance** | Business Critical, 4 vCore, HA nativa, geo-replication | Incluso in Azure IaaS |
| **PostgreSQL Flexible Server** | Zone-Redundant HA, failover <2 min | Incluso in Azure IaaS |
| **VM generiche** | Zimbra (D4s v5), Odoo (E8s v5), Gitea (D4s v5), MediaWiki (D2s v5), IIS Pondus (D4s v5 x2) | Incluso in Azure IaaS |

> **Breakdown HPC Anno 1:**
> ```
> Compute (16 nodi HBv4):
>   On-demand (30% carico): 16 x 1.500h x ~€4,80/h = €115.200
>   Spot (70% carico): 16 x 3.500h x ~€1,45/h =  €81.200
>   Subtotale: €196.400
> Storage (NetApp Files 50TB premium): €27.000
> Egress (50TB/anno): €10.000
> Ciclo di setup iniziale + benchmark: €26.600
>   TOTALE arrotondato: €230.000
> ```

### 6.3 Servizi SaaS (pass-through)

| Servizio | Copertura | Costo Annuo |
|---|---|---|
| Microsoft 365 E5 | 50 utenti | €33.000 |
| Microsoft Entra ID Premium P2 | 50 utenti | Incluso in E5 |
| Microsoft Defender for Endpoint | 50 device | Incluso in E5 |
| Microsoft Purview | Compliance, ISO 27001 | €5.000 |
| Microsoft Sentinel | SIEM (opzionale) | €5.000 |

### 6.4 Esclusioni Hardware

- Switch di rete, cablaggio, access point WiFi (gia esistenti in sede Thema)
- Laptop e device utente (ogni dipendente ha gia un laptop aziendale)
- Gruppo di continuita (UPS) per la sede (gestito da TP Group)
- Infrastruttura di raffreddamento (non necessaria — zero HW on-premise)

---

## 7. Attivita Richieste

### 7.1 Fasi del Progetto (12 settimane)

```
      Sett. 1   2   3   4   5   6   7   8   9  10  11  12
1. Prep        XX  XX
2. Identity        XX  XX
3. Collab              XX  XX  XX
4. App                    XX  XX  XX  XX  XX
5. HPC               XX  XX  XX  XX  XX
6. Security                     XXXXXXXXXXXXXXXX
7. Governance                              XX  XX
8. Collaudo                                   XX  XX  XX
9. Chiusura                                           XX  XX
```

### 7.2 Dettaglio Attivita

#### Fase 1 — Preparazione (Sett. 1-2)
- Assessment approfondito infrastrutture Thema e Pro Studio
- Attivazione contratti Microsoft M365/Azure
- Nomina security team e referente interno TP Group

#### Fase 2 — Identity First (Sett. 2-3)
- Attivazione Entra ID Premium P2
- Sincronizzazione Samba4 AD + Google Identity
- Provisioning identita per 50 utenti
- Configurazione MFA e Conditional Access base

#### Fase 3 — Migrazione Collaborazione (Sett. 3-5)
- Migrazione posta: Zimbra (30 utenti) + Gmail (20 utenti) -> Exchange Online
- Migrazione file: NextCloud + Google Drive -> OneDrive/SharePoint
- Configurazione DLP, Safe Attachments/Links
- Adozione Teams per tutti i 50 utenti

#### Fase 4 — Migrazione Applicazioni (Sett. 4-8)
- Pondus: P2V su Azure VM + Azure SQL MI (Business Critical, geo-replication)
- Odoo: VM dedicata (E8s v5) + Azure Database for PostgreSQL Flexible Server Zone-Redundant HA (failover automatico <2 min)
- Gitea, MediaWiki: deploy su Azure VM
- Azure Data Box per migrazione 40TB bulk

#### Fase 5 — HPC (Sett. 3-6)
- Provisioning Azure HBv4 (16 nodi elastici)
- Configurazione CycleCloud + Slurm
- VPN HPC dedicata (FortiGate -> Azure VPN Gateway)
- Migrazione carichi HPC da on-prem a cloud

#### Fase 6 — Sicurezza & DR (Sett. 5-8)
- **6.1 Identity & Endpoint:** Conditional Access avanzato, PIM, Defender for Endpoint, Attack Simulation Training
- **6.2 Rete & Infrastruttura:** FortiGate 100F HA, Azure Firewall, NSG, VPN S2S, VLAN
- **6.3 Backup & DR:** Azure Backup, Site Recovery, test RPO/RTO
- **6.4 Audit & Compliance:** Log Analytics, Purview, dashboard Grafana

#### Fase 7 — Governance (Sett. 8-9)
- Azure Policy, RBAC, PIM, Access Reviews
- Automazione IaC (Terraform, GitHub Actions, self-healing)

#### Fase 8 — Collaudo & Go-Live (Sett. 9-11)
- Penetration test esterno (full-scope)
- Test RPO/RTO completi
- Formazione utenti (M365 adoption, security awareness)
- Cut-over finale

#### Fase 9 — Chiusura (Sett. 11-12)
- Dismissione infrastruttura on-premise Thema (VMware, SAN, QNAP, FG-60F)
- Chiusura account Google Workspace Pro Studio
- Handover al team IT TP Group
- Documentazione as-built e runbook operativi

### 7.3 Attivita Escluse

- Sviluppo software o modifiche al codice di Pondus
- Migrazione di workload non identificati nell'analisi AS-IS
- Supporto help-desk continuativo oltre il pacchetto post-go-live

---

## 8. Documentazione Rilasciata

Al termine del progetto, 5Stack rilascera la seguente documentazione:

| Documento | Descrizione |
|---|---|
| **Documento di Architettura (HLD)** | Schema infrastruttura target, scelte architetturali, diagramma di rete |
| **Documento di Configurazione (LLD)** | Dettaglio configurazioni Azure, M365, FortiGate |
| **Runbook Operativi** | Procedure di gestione ordinaria, backup/restore, failover DR |
| **Piano di Test** | Report test funzionali, test di carico, penetration test |
| **Documento di Migrazione** | Mappatura servizi AS-IS -> TO-BE, cronologia migrazioni |
| **Manuale Utente** | Guida all'uso di M365, Teams, OneDrive per i dipendenti |
| **Documento di Disaster Recovery** | Procedure RPO/RTO, failover, contatti di emergenza |
| **Infrastructure as Code (Terraform)** | Repository Git con tutto il codice IaC |

---

## 9. Termini e Condizioni

### Validita dell'Offerta
La presente offerta e valida **30 giorni** dalla data di emissione.

### Modalita di Pagamento
- **50%** all'avvio del progetto
- **25%** al completamento della Fase 5 (HPC)
- **25%** al completamento della Fase 9 (Chiusura)

I costi di infrastruttura (pass-through Azure, M365, FortiGate) sono fatturati al costo netto senza markup e pagabili a 30 giorni data fattura.

### Garanzia
5Stack garantisce la corretta configurazione e funzionalita di tutti i sistemi oggetto di installazione e migrazione per **90 giorni** dal go-live, inclusa assistenza su eventuali anomalie emerse.

### Risoluzione
In caso di recesso anticipato, TP Group corrispondera a 5Stack le prestazioni effettivamente erogate fino alla data di recesso, calcolate in proporzione al forfait concordato.

### Privacy e Riservatezza
5Stack si impegna a trattare tutti i dati aziendali di TP Group con la massima riservatezza, firmando specifico NDA (Non-Disclosure Agreement) prima dell'avvio dei lavori.

---

## 10. Quotazione Economica

### 10.1 Infrastruttura Anno 1 (pass-through)

| Voce | Costo |
|---|---|
| Microsoft 365 E5 (50 utenti) | €33.000 |
| Azure IaaS/PaaS (VM, SQL MI, AKS, PostgreSQL) | €40.000 |
| Azure Firewall Standard + NSG + DDoS Protection | €10.000 |
| Azure Backup + Site Recovery | €8.000 |
| HPC — Azure HBv4 spot + on-demand (16 nodi) | €230.000 |
| HPC Storage — Azure NetApp Files / Managed Lustre | €27.000 |
| HPC Egress — dati in uscita verso sede | €10.000 |
| *(di cui compute: €196.400 / storage: €27.000 / egress: €10.000 / setup: €26.600)* | |
| Azure Support Plan Standard (risposta <1h) | €12.000 |
| Azure Bastion | €2.000 |
| Microsoft Purview (compliance ISO 27001) | €5.000 |
| FortiGate 100F HA pair (copre 3 anni) | €15.000 |
| Microsoft Sentinel (opzionale) | €5.000 |
| **Subtotale** | **€397.000** |
| Fondo imprevisti (~13%) | **€53.000** |
| **TOTALE INFRASTRUTTURA** | **€450.000** |

### 10.2 Costi Ricorrenti Fissi (da Anno 2)

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

> **HPC variabile**: compute, storage ed egress a consumo — stimato €35.000–€125.000/anno

### 10.3 Servizi 5Stack — Costruzione (una tantum)

**€120.000 forfait** per 12 settimane di progetto, team dedicato:

| Ruolo | Tariffa | Giorni | Importo |
|---|---|---|---|
| Project Manager | €650/gg | 26gg | €16.900 |
| Sistemisti Senior (x4) | €500/gg | 161gg | €80.500 |
| Trasferta | — | — | €1.100 |
| **Costo effettivo** | | 187gg | **€98.500** |
| **Forfait 5Stack** | | | **€120.000** |
| *Margine* | | | *18%* |

### 10.4 Pacchetti Post-Go-Live (opzionali)

Tutti i pacchetti includono **6 mesi di formazione in sede (21 giornate)**:

| # | Pacchetto | Formazione | Presenza | Assistenza | **Prezzo** |
|---|---|---|---|---|---|
| 1 | **Basic** | 6 mesi (21gg) | Solo formazione | Remoto, SLA 24h | **€18.000** |
| 2 | **Standard** | 6 mesi (21gg) | 2gg/mese (24gg/anno) | Remoto + in sede, SLA 8h | **€33.000** |
| 3 | **Plus** | 6 mesi (21gg) | 2gg/mese A1, 1gg/mese A2 | 2 anni, SLA 8h | **€46.000** |
| 4 | **Premium** | 12 mesi (42gg) | 2gg/mese (48gg) | 24/7, SLA 4h | **€58.000** |

### 10.5 Scenari Completi

#### Scenario Standard (consigliato)

| Voce | Anno 1 | Anno 2 | Anno 3+ |
|---|---|---|---|
| Infrastruttura (pass-through) | €450.000 | €95.000* | €95.000/anno* |
| Costruzione 5Stack | €120.000 | — | — |
| Pacchetto Standard | €33.000 | — | — |
| **TOTALE** | **€603.000** | **€95.000*** | **€95.000/anno*** |

* HPC variabile escluso (compute, storage, egress a consumo: €35k–€125k/anno)

### 10.6 Analisi TCO — 3 Anni

Confronto tra Full Cloud (proposta) e potenziamento on-premise tradizionale.

| Voce | Full Cloud | On-premise potenziato |
|---|---|---|
| **Anno 1** — Infrastruttura | €450.000 | €742.000* |
| **Anno 1** — Servizi professionali | €153.000 | €50.000 |
| **Anno 2** | €95.000 | €297.000** |
| **Anno 3** | €95.000 | €297.000** |
| **Totale 3 anni (fisso)** | **€793.000** | **€1.386.000** |
| **HPC variabile (3 anni)** | ~€150.000 | Incluso |
| **TOTALE 3 ANNI** | **~€943.000** | **~€1.386.000** |

**Risparmio TCO: ~€443.000 (~32%) in 3 anni**

> **Note TCO:**
> *On-premise Anno 1: nuovo rack + raffreddamento €50k, UPS €20k, 3 server €90k, SAN 50TB €60k, 8 nodi HPC €270k (3yr lease), FortiGate HA €15k, backup enterprise €30k, 2 sistemisti €120k, VMware €15k, M365 E3 €22k
> **On-premise Anno 2-3: 2 sistemisti €120k, manutenzione HW €25k, elettricità/raffreddamento €30k, ammortamento HPC €90k, M365 E3 €22k, VMware €10k

#### Vantaggi intangibili del Full Cloud (non quantificati nel TCO)

| Vantaggio | Beneficio |
|---|---|
| **ISO 27001 compliance** | Zero investimenti aggiuntivi per certificazione |
| **Zero rischio HW** | Nessuna sostituzione server, nessun guasto componenti |
| **Scalabilita immediata** | Nuovo nodo in 5 minuti vs 4-6 settimane per ordine HW |
| **DR multi-region** | Incluso, nessun costo aggiuntivo per sito secondario |
| **Team IT ridotto** | 6 risorse invece di 8 grazie ad automazione e cloud |
| **Energy saving** | Zero elettricita/raffreddamento per server in sede |

#### Confronto Pacchetti

| | Base | Basic | Standard | Plus | Premium |
|---|---|---|---|---|---|
| **Anno 1** | €570.000 | €588.000 | €603.000 | €616.000 | €628.000 |
| **Anno 2*** | €95.000 | €95.000 | €95.000 | €95.000 | €95.000 |
| **2 anni*** | €665.000 | €683.000 | €698.000 | €711.000 | €723.000 |

---

## Appendice A — Architettura Zero Trust: 9 Strati

| # | Strato | Soluzione |
|---|---|---|
| 1 | **Identity** | Entra ID Premium P2, MFA, Conditional Access, PIM, Access Reviews |
| 2 | **Endpoint** | Defender for Endpoint (EDR, ASR, threat hunting) |
| 3 | **SaaS (M365)** | M365 Defender, Anti-phishing, Safe Links/Attachments, DLP, CASB |
| 4 | **IaaS (Azure)** | Azure Firewall, NSG, JIT VM Access, Key Vault, Disk Encryption |
| 5 | **Sede** | FortiGate 100F HA, VPN S2S, VLAN segmentazione |
| 6 | **Backup & DR** | Azure Backup, Site Recovery, SQL MI geo-replication |
| 7 | **Audit & Compliance** | Log Analytics, Purview, ISO 27001 |
| 8 | **Governance** | Azure Policy, RBAC, PIM, Access Reviews |
| 9 | **Training** | Attack Simulation Training, security awareness |

## Appendice B — Roadmap di Progetto (Gantt)

```
      Settimana      1   2   3   4   5   6   7   8   9  10  11  12
Fase 1: Preparazione            [====]
Fase 2: Identity                    [====]
Fase 3: Collaborazione                  [==========]
Fase 4: Applicazioni                         [==================]
Fase 5: HPC                           [==================]
Fase 6: Sicurezza & DR                              [========================]
Fase 7: Governance                                       [========]
Fase 8: Collaudo & Go-Live                                    [==============]
Fase 9: Chiusura                                                       [========]
```

## Appendice C — Tabella RASCI

| Attivita | PM | Sist. Infra | Net & Sec | Cloud & App | Doc & Integ | Cliente |
|---|---|---|---|---|---|---|
| Assessment AS-IS | A | R | R | R | C | C |
| Progettazione architettura | A | C | C | R | C | I |
| Setup Entra ID / Identity | A | C | C | R | I | C |
| Migrazione posta | A | C | I | R | R | C |
| Migrazione file | A | C | I | R | R | C |
| Migrazione applicazioni | A | I | I | R | C | C |
| Setup HPC Azure | A | R | C | C | I | I |
| Configurazione FortiGate | A | I | R | I | I | C |
| Backup & DR | A | R | C | C | I | C |
| Penetration test | A | I | R | C | I | I |
| Formazione utenti | A | C | C | C | R | C |
| Documentazione as-built | A | C | C | C | R | I |
| Dismissione on-prem | A | R | C | C | C | I |

*R = Responsible | A = Accountable | C = Consulted | I = Informed*

---

*Documento v1.0 — Proposta finale per TP Group srl.*
*Ogni sezione e stata redatta secondo i requisiti del capitolato Learning by Project.*
