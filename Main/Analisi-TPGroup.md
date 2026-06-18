# Analisi AS-IS — Situazione attuale TP Group srl

**Data:** 12/06/2026  
**Redatto da:** Lorenzo Airoldi (PM)  
**Versione:** 1.0

---

## 1. Thema Consulting

Fondata fine anni '90, 30 dipendenti, sede di proprietà a Torino. Dispone di un data center on-premise con 2 rack saturi.

### 1.1 Virtualizzazione
| Elemento | Dettaglio |
|----------|-----------|
| Piattaforma | VMware (2 host) |
| Storage | SAN NetApp 24 TB |
| Note | Rack saturi, condizionamento al limite |

### 1.2 Backup
| Elemento | Dettaglio |
|----------|-----------|
| Soluzione | NAS QNAP 32 TB |
| Tipo | On-premise |
| Note | Nessun backup delocalizzato |

### 1.3 Firewall / Networking
| Elemento | Dettaglio |
|----------|-----------|
| Firewall | Fortigate FG-60F |
| Note | Valutare se adeguato per 50 utenti + VPN |

### 1.4 Cluster HPC
| Elemento | Dettaglio |
|----------|-----------|
| Nodi | 8 nodi fisici Linux |
| Interconnessione | Infiniband |
| Note | Obiettivo: +8 nodi entro 2 anni (totale 16) |

### 1.5 Servizi IT

| Servizio | Soluzione | Piattaforma |
|----------|-----------|-------------|
| Email/Calendario | Zimbra | VM Linux |
| Produttività | MS365 / LibreOffice | Multi-piattaforma |
| Video/IM | MS Teams | SaaS |
| Autenticazione | Active Directory (Samba4) | VM Linux |
| File Server | NextCloud | VM Linux |
| Dev/Versioning | Gitea | VM Linux |
| Contabilità | Profis | On-premise, Windows |
| CRM | SugarCRM | VM Linux |
| Timesheet | Kimai | VM Linux |
| Documentazione | MediaWiki | VM Linux |
| App proprietaria | Pondus (IIS + MSSQL) | HW fisico dedicato |

---

## 2. Pro Studio

Startup fondata nel 2023, 20 dipendenti. Nessuna infrastruttura IT interna, tutto su cloud.

### 2.1 Cloud / SaaS in uso

| Servizio | Soluzione | Tipo |
|----------|-----------|------|
| Email/Calendario | Google Workspace | SaaS |
| Produttività | Google Workspace | SaaS |
| Video/IM | Google Meet | SaaS |
| Autenticazione | Google Authenticator | SaaS |
| File Server | Google Drive | SaaS |
| Dev/Versioning | Bitbucket | SaaS |
| Contabilità | Odoo | SaaS |
| CRM | Odoo | SaaS |
| Timesheet | Odoo | SaaS |
| Documentazione | MediaWiki | GCP (IaaS) |
| Backup | Google Cloud (GCP) | Cloud |

### 2.2 Infrastruttura
| Elemento | Dettaglio |
|----------|-----------|
| Sede fisica | Affitto in scadenza (non rinnovato) |
| Data center | Nessuno |
| HW | Nessun server on-premise |

---

## 3. Criticità emerse

| # | Criticità | Impatto |
|:-:|-----------|---------|
| 1 | **Rack saturi** — Data center Thema non espandibile | Impossibile aggiungere HW on-premise senza liberare spazio |
| 2 | **Condizionamento al limite** — Raffreddamento insufficiente | Rischio surriscaldamento, vincola densità di potenza |
| 3 | **Servizi duplicati** — 7 aree con soluzioni diverse | Complessità gestionale, costi mantenimento doppi |
| 4 | **Backup solo on-premise** — Nessuna copia off-site | Non conforme ISO/IEC 27001 (DR) |
| 5 | **Nessuna ridondanza** — Singolo firewall, singolo NAS | Single point of failure su componenti critici |
| 6 | **Pondus su HW fisico** — Server dedicato non ridondato | Rischio downtime applicazione critica |
| 7 | **Autenticazione disomogenea** — Samba4 vs Google Auth | Nessun SSO, doppia gestione utenti |

---

## 4. Servizi duplicati (da unificare)

| Area | Thema | Pro Studio | Azione necessaria |
|------|-------|------------|-------------------|
| Email/Calendario | Zimbra | Google Workspace | Scegliere piattaforma unica |
| File Server | NextCloud | Google Drive | Unificare storage |
| CRM | SugarCRM | Odoo | Scegliere piattaforma unica |
| Timesheet | Kimai | Odoo | Scegliere piattaforma unica |
| Documentazione | MediaWiki (on-prem) | MediaWiki (GCP) | Unificare su singola istanza |
| Dev/Versioning | Gitea | Bitbucket | Scegliere piattaforma unica |
| Contabilità | Profis | Odoo | Integrazione o consolidamento |

---

## 5. Vincoli progettuali

- **50 dipendenti** (30+20) nella sede Thema a Torino
- **VDI escluso** — ogni dipendente ha laptop aziendale
- **ISO/IEC 27001** — business continuity, DR, sicurezza accessi, cifratura, VPN
- **Budget HPC:** 250.000€ + IVA (on-premise/hosting) oppure 220.000€ + IVA (cloud/noleggio)
- **Costo sistemista:** 500€/giorno | **PM:** 650€/giorno
