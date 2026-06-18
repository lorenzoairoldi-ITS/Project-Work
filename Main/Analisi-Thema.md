# Analisi AS-IS — Thema Consulting

**Data:** 12/06/2026  
**Redatto da:** 5Stack  
**Versione:** 1.0

---

## 1. Profilo aziendale

| Dato | Valore |
|------|--------|
| Nome | Thema Consulting |
| Fondazione | Fine anni '90 |
| Settore | Automotive / Aerospace |
| Dipendenti | 30 |
| Sede | Torino (di proprietà) |
| Data center | On-premise, 2 rack saturi |
| Cluster HPC | 8 nodi fisici Linux + Infiniband |
| App proprietaria | Pondus (IIS + MSSQL, HW fisico) |

---

## 2. Infrastruttura virtualizzazione

### 2.1 VMware (2 host) + SAN NetApp 24 TB

**Cosa usano:** Due server fisici con hypervisor VMware collegati a uno storage centralizzato NetApp da 24 TB tramite SAN (Storage Area Network).

**Perché VMware:**
- Piattaforma enterprise matura, standard de facto nel settore industriale/manifatturiero da oltre 20 anni
- Supporto per alta affidabilità (HA), vMotion (migrazione a caldo di VM), DRS (bilanciamento carico)
- Ecosistema ampio di partner e strumenti di backup compatibili (Veeam, CommVault, ecc.)
- Scelta conservativa tipica di aziende storiche: una volta adottato VMware, la migrazione verso altri hypervisor è costosa e rischiosa

**Perché SAN NetApp:**
- Storage centralizzato ad alte prestazioni, ideale per carichi di lavoro virtualizzati
- NetApp offre funzionalità di snapshot, deduplica e compressione inline
- Affidabilità enterprise con ridondanza (controller HA, disk shelf, percorsi fiber channel multipli)
- 24 TB di capacità: spazio adeguato per le VM esistenti, ma da valutare per la crescita futura (+20 utenti Pro Studio)

**Contesto:**
Tutte le VM aziendali (Zimbra, NextCloud, SugarCRM, Kimai, MediaWiki, Samba4 AD, Gitea) risiedono su questa infrastruttura. È il cuore pulsante dell'IT di Thema.

---

## 3. Backup

### 3.1 NAS QNAP 32 TB (on-premise)

**Cosa usano:** Un NAS QNAP da 32 TB collegato in rete locale che riceve i backup delle VM e dei dati via rete.

**Perché QNAP:**
- Soluzione storage economica rispetto a un sistema di backup enterprise (tipo Data Domain o StoreOnce)
- Integrazione nativa con Veeam (se usano Veeam) o funzionalità di backup built-in
- 32 TB sufficienti per i backup correnti

**Criticità:**
- Backup interamente on-premise → nessuna copia off-site → violazione dei requisiti ISO/IEC 27001 per il disaster recovery
- Singolo NAS → single point of failure: se il NAS si guasta o viene colpito da ransomware, i backup sono persi
- QNAP è un prodotto consumer/prosumer, non enterprise: uptime, supporto e prestazioni inferiori rispetto a soluzioni dedicate

---

## 4. Firewall e rete

### 4.1 Fortigate FG-60F

**Cosa usano:** Un firewall Fortigate modello FG-60F.

**Perché Fortigate:**
- Marchio leader nel mercato firewall, molto diffuso in Italia per PMI
- Funzionalità NGFW (Next-Generation Firewall): IPS, antivirus, filtro web, applicazioni, SSL inspection
- Gestione centralizzata via FortiManager / FortiCloud
- Il modello FG-60F è un entry-level per piccole sedi o filiali

**Criticità:**
- FG-60F è dimensionato per ≤30 utenti. Con l'arrivo dei 20 dipendenti Pro Studio (totale 50), il throughput e le connessioni concorrenti potrebbero essere insufficienti
- Non ha ridondanza (singolo firewall): se si guasta, l'intera connettività è persa
- Il carico VPN (LAN-to-LAN + Client-to-LAN per remote work) potrebbe saturarlo

---

## 5. Cluster HPC

### 5.1 8 nodi fisici Linux + Infiniband

**Cosa usano:** 8 server fisici dedicati al calcolo ad alte prestazioni, interconnessi tramite rete Infiniband per bassa latenza e alta banda.

**Perché HPC on-premise:**
- Il settore automotive/aerospace richiede simulazioni, analisi CFD (Computational Fluid Dynamics), modellazione FEM (Finite Element Method), rendering
- Questi carichi richiedono latenza ultra-bassa e bandwidth elevato tra i nodi → Infiniband è la scelta obbligata
- I dati di lavoro sono voluminosi (terabyte per simulazione): tenerli in sede elimina tempi di trasferimento su cloud
- Budget già stanziato per espansione: +8 nodi (totale 16) entro 2 anni

**Specifiche per i nuovi nodi (dal capitolato):**
- 2 CPU, ≥24 core per CPU, clock elevato
- 8 GB RAM per core (es. 48 core → 384 GB RAM)
- No GPU
- 1,6 TB scratch SSD per nodo
- Linux OS
- Infiniband 100/200 Gbps

**Contesto:**
Il cluster è usato esclusivamente da personale interno Thema (ingegneri, ricercatori). Le simulazioni possono durare ore o giorni. Non ci sono VM o container sugli HPC.

---

## 6. Servizi IT

### 6.1 Email / Calendario — Zimbra (VM Linux)

**Cosa usano:** Zimbra Collaboration Suite su VM Linux.

**Perché Zimbra:**
- Soluzione open source matura con client web, supporto IMAP/SMTP, calendario condiviso
- Costo inferiore rispetto a Exchange Online o Google Workspace per 30 utenti
- Controllo completo dei dati (dati in sede, non su cloud di terzi)
- Scelta tipica di aziende con competenze Linux interne

**Criticità:**
- Richiede manutenzione (patch, backup, aggiornamenti)
- Non ha la stessa integrazione di Exchange con Outlook o Teams
- Con 50 utenti (post-fusione) va ridimensionata

### 6.2 Produttività — MS365 / LibreOffice

**Cosa usano:** Doppia soluzione: Microsoft 365 (licenze) + LibreOffice open source.

**Perché:**
- MS365 per compatibilità con clienti/fornitori (formati .docx, .xlsx, .pptx)
- LibreOffice usato internamente per ridurre i costi di licenza
- Scelta ibrida per ottimizzare il budget

**Criticità:**
- Doppia gestione, problemi di compatibilità occasionali
- Incertezza su quale piattaforma standardizzare post-fusione

### 6.3 Video / IM — MS Teams

**Cosa usano:** Microsoft Teams per messaggistica e videoconferenze.

**Perché Teams:**
- Integrato con MS365
- Standard in molti clienti automotive/aerospace
- Funzionalità di canali, team, condivisione file

### 6.4 Autenticazione — Active Directory (Samba4, VM Linux)

**Cosa usano:** Samba4 su VM Linux che funge da controller di dominio Active Directory.

**Perché Samba4 invece di Windows Server AD:**
- Evita il costo delle licenze Windows Server per l'AD
- Samba4 è compatibile con il protocollo AD di Microsoft (join a dominio, GPO, Kerberos)
- Scelta da azienda con competenze Linux/hardcore

**Criticità:**
- Samba4 non implementa tutte le funzionalità di Windows AD (es. Group Policy avanzate, PowerShell remoting, AD Federation Services)
- Complessità di gestione maggiore
- Con l'arrivo dei 20 utenti Pro Studio, va valutata l'unificazione con Google Authenticator / Workspace

### 6.5 File Server — NextCloud (VM Linux)

**Cosa usano:** NextCloud su VM Linux per condivisione file e sincronizzazione.

**Perché NextCloud:**
- Alternativa open source a Google Drive / Dropbox / OneDrive
- Sincronizzazione client multi-piattaforma (Windows, Mac, Linux, mobile)
- Controllo completo dei dati (on-premise)
- Funzionalità di condivisione con link, versioning, crittografia lato server

**Criticità:**
- I 20 utenti Pro Studio arrivano con Google Drive come abitudine
- Va valutata migrazione dei dati e accettazione utenti

### 6.6 Dev / Versioning — Gitea (VM Linux)

**Cosa usano:** Gitea, piattaforma Git self-hosted leggera.

**Perché Gitea:**
- Alternativa open source e leggera a GitHub/GitLab/Bitbucket
- Hosting on-premise per codice sensibile (progetti automotive/aerospace)
- Basso consumo di risorse (golang, funziona su VM piccola)

**Criticità:**
- Pro Studio usa Bitbucket (SaaS Atlassian): migrazione o mantenimento di due piattaforme?

### 6.7 Contabilità — Profis (on-premise, Windows)

**Cosa usano:** Profis, software contabile su macchina Windows on-premise.

**Perché Profis:**
- Software contabile storico, diffuso in Italia
- Dati finanziari sensibili: tenerli on-premise garantisce controllo
- Integrazione con banche, fatturazione elettronica, dichiarativi fiscali

**Criticità:**
- Pro Studio usa Odoo (SaaS) per contabilità: due sistemi da unificare
- Profis richiede Windows Server, con relative licenze

### 6.8 CRM — SugarCRM (VM Linux)

**Cosa usano:** SugarCRM su VM Linux, self-hosted.

**Perché SugarCRM:**
- CRM open source self-hosted, personalizzabile
- Dati commerciali sensibili on-premise
- Costi di licenza inferiori rispetto a Salesforce / Dynamics 365

**Criticità:**
- Pro Studio usa Odoo come CRM: due CRM da consolidare in uno

### 6.9 Timesheet — Kimai (VM Linux)

**Cosa usano:** Kimai, software di timesheet open source su VM Linux.

**Perché Kimai:**
- Leggero, semplice, open source
- Tracciamento ore per progetti committenti (tipico di consulenza)
- Self-hosted per controllo dati

**Criticità:**
- Pro Studio usa il modulo timesheet di Odoo

### 6.10 Documentazione — MediaWiki (VM Linux)

**Cosa usano:** MediaWiki su VM Linux on-premise.

**Perché MediaWiki:**
- Standard per documentazione tecnica interna
- Wiki semantico con categorie, template, allegati
- Stessa identica piattaforma usata da Pro Studio (ma su GCP)

**Criticità:**
- Pro Studio ha MediaWiki su GCP: due istanze separate. Unificazione semplice (stesso software) ma va pianificata

---

## 7. Applicazione proprietaria: Pondus

### 7.1 Pondus (IIS + MSSQL, HW fisico)

**Cosa usano:** Applicazione proprietaria scritta in C# su IIS (Internet Information Services) con database Microsoft SQL Server. Tutto su server fisico dedicato.

**Perché su HW fisico:**
- Probabilmente è stata sviluppata anni fa e messa in produzione su HW fisico, senza mai migrarla su VM
- Requisiti di performance / IOPS che all'epoca sembravano richiedere HW dedicato
- Inerzia tecnica: "se funziona non si tocca"

**Criticità:**
- **Nessuna ridondanza:** server singolo → se si guasta, l'applicazione è offline
- **Nessuna flessibilità:** difficile scalare, aggiornare o fare manutenzione
- **Obsolescenza HW:** il server fisico prima o poi va sostituito
- Obiettivo: potenziamento e distribuzione a un'utenza più ampia (da 30 a 50+)

---

## 8. Obiettivi strategici di Thema Consulting

| Obiettivo | Dettaglio |
|-----------|-----------|
| **Business Continuity** | Ridondanza sistemi critici, UPS adeguato, connettività ridondata |
| **Disaster Recovery** | Backup delocalizzato, tempi di ripristino garantiti (RPO/RTO) |
| **Sicurezza ISO 27001** | Cifratura storage, VPN, controllo accessi, audit trail |
| **Crescita HPC** | +8 nodi (totale 16), Infiniband 100/200 Gbps |
| **Potenziamento Pondus** | Scalabilità, ridondanza, più utenti |
| **Unificazione IT** | Assorbimento dei 20 dipendenti Pro Studio con minimo impatto |

---

## 9. Vincoli fisici

| Vincolo | Impatto |
|---------|---------|
| **2 rack saturi** | Impossibile aggiungere HW on-premise senza rimuovere qualcosa |
| **Condizionamento al limite** | Impossibile aumentare la densità di potenza nei rack |
| **UPS 10 minuti** | Autonomia insufficiente per spegnimento controllato di tutti i sistemi |
| **No VDI** | Ogni utente ha laptop aziendale |
| **50 persone in sede** | Raddoppio del carico su rete locale, firewall, server |

---

## 10. Schema riassuntivo infrastruttura

```
┌─────────────────────────────────────────────────────────────┐
│                    THEMA CONSULTING                          │
│                        30 utenti                             │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────────┐  ┌──────────────┐  ┌────────────┐  │
│  │  VMware (2 host)    │  │  HPC Cluster  │  │   Pondus   │  │
│  │  + SAN NetApp 24TB  │  │  8 nodi Linux │  │ IIS+MSSQL  │  │
│  │                     │  │  + Infiniband │  │ HW fisico  │  │
│  │  VM: Zimbra         │  │               │  │            │  │
│  │  VM: NextCloud      │  │               │  │            │  │
│  │  VM: SugarCRM       │  │               │  │            │  │
│  │  VM: Kimai          │  │               │  │            │  │
│  │  VM: MediaWiki      │  │               │  │            │  │
│  │  VM: Samba4 AD      │  │               │  │            │  │
│  │  VM: Gitea          │  │               │  │            │  │
│  └─────────────────────┘  └──────────────┘  └────────────┘  │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐   │
│  │  Firewall    │  │  NAS QNAP    │  │  UPS 10'         │   │
│  │  Fortigate   │  │  32 TB       │  │                  │   │
│  │  FG-60F      │  │  Backup on-  │  │  Condiz. limite  │   │
│  │              │  │  premise     │  │                  │   │
│  └──────────────┘  └──────────────┘  └──────────────────┘   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

*Fine analisi AS-IS Thema Consulting — 12/06/2026*
