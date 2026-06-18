# Progetto Infrastruttura IT – TP Group srl
**Unità Formativa:** Learning by project | **Docente:** Luca Robaldo

---

## Contesto

Due aziende del settore automotive/aerospace si fondono per creare **TP Group srl**:

- **Thema Consulting** – fondata fine anni '90, 30 persone, sede di proprietà a Torino, data center on-premise (2 rack saturi), cluster HPC, applicazione proprietaria *Pondus*
- **Pro Studio** – startup 2023, 20 persone, nessuna infrastruttura IT interna, tutto su cloud (Google Workspace, GCP, SaaS)

Con la fusione, **i dipendenti di Pro Studio si trasferiscono nella sede di Thema Consulting**. Il contratto di affitto di Pro Studio non viene rinnovato.

---

## Cosa dobbiamo fare

Redigere un **documento di offerta tecnico/economica** per la convergenza e il potenziamento dei sistemi IT delle due aziende. Il documento deve rispettare i requisiti elencati nel capitolato.

---

## Situazione attuale (da integrare)

| Area | Thema Consulting | Pro Studio |
|---|---|---|
| Virtualizzazione | VMware (2 host) + SAN NetApp 24TB | Nessuna |
| Backup | NAS QNAP 32TB on-premise | Google Cloud (GCP) |
| Firewall | Fortigate FG-60F | N/A |
| Cluster HPC | 8 nodi fisici Linux + Infiniband | N/A |
| Email/Cal | Zimbra (VM Linux) | Google Workspace |
| Produttività | MS365 / LibreOffice | Google Workspace |
| Video/IM | MS Teams | Google Meet |
| Autenticazione | Active Directory (Samba4, VM Linux) | Google Authenticator |
| File Server | NextCloud (VM Linux) | Google Drive |
| Dev/Versioning | Gitea | Bitbucket (SaaS) |
| Contabilità | Profis (on-premise, Windows) | Odoo (SaaS) |
| CRM | SugarCRM (VM Linux) | Odoo (SaaS) |
| Timesheet | Kimai (VM Linux) | Odoo (SaaS) |
| Documentazione | MediaWiki (VM Linux) | MediaWiki (GCP) |
| App proprietaria | **Pondus** (IIS + MSSQL, hw fisico) | N/A |

---

## Vincoli da rispettare

### Tecnologici (ISO/IEC 27001)
- **Business continuity**: ridondanza dei sistemi critici e delle connettività
- **Disaster recovery**: backup delocalizzati, servizi clienti sempre fruibili
- **Sicurezza accessi**: cifratura storage, protocolli sicuri, VPN (LAN-to-LAN, Client-to-LAN, Hybrid cloud)

### Logistici
- Tutti i 50 dipendenti (30 + 20) lavoreranno nella sede Thema a Torino
- Il data center esistente **non può essere espanso** (rack saturi, condizionamento al limite)
- Ogni dipendente ha un laptop aziendale; **non si usano soluzioni VDI**

---

## Obiettivi futuri da includere nell'offerta

### Cluster HPC
- Aggiungere **8 nuovi nodi** entro 2 anni (totale 16)
- Nodi fisici (no VM, no container) – uso esclusivo personale interno
- Rete doppia: Ethernet + **Infiniband 100/200 Gbps**
- Specifiche per nodo: 2 CPU, ≥24 core, clock elevato, 8 GB RAM/core, no GPU, Linux, 1,6 TB scratch
- Budget: **250.000€ + IVA** (on-premise/hosting) oppure **220.000€ + IVA** (cloud/noleggio)

### Applicazione Pondus
- Potenziamento e distribuzione a un'utenza più ampia

---

## Struttura del documento da consegnare

Il documento deve contenere **obbligatoriamente**:

1. **Pagina introduttiva**
2. **Indice**
3. **Presentazione del team** (chi siete, servizi offerti, storia)
4. **Introduzione** – sintesi dell'offerta
5. **Assunzioni** – prerequisiti per l'erogazione del progetto
6. **Descrizione fornitura hardware** – materiali, esclusioni, note
7. **Attività richieste** – installazione HW/SW, test funzionali, esclusioni, note
8. **Documentazione** – cosa verrà rilasciato
9. **Termini e condizioni** (se presenti)
10. **Quotazione economica**

**Nice to have (bonus)**:
- Gantt di progetto
- Tabella RASCI

---

## Criteri di valutazione

La soluzione proposta sarà valutata rispetto a queste dimensioni:

| Dimensione | Opzioni da valutare |
|---|---|
| Erogazione del servizio | On-premise / Cloud / Hosting co-location (min. 3 anni) |
| Tipo di server | Fisico / Virtuale / Container |
| Sistema operativo | Windows Server / Linux |
| Tipo di soluzione | Open source / Commerciale |
| Proprietà beni IT | A noleggio / Di proprietà |
| Gestione IT | Personale interno / Outsourcing |
| Pricing | Outsourcing / Personale interno |

---

## Costi di riferimento per il piano economico

- Tecnico sistemista (≥5 anni exp): **500€/giorno**
- Project Manager: **650€/giorno**

---

## Checklist rapida

- [ ] Scegliere come unificare i servizi duplicati (email, file server, CRM, documentazione, ecc.)
- [ ] Decidere dove collocare i sistemi (on-premise, cloud, ibrido) tenendo conto dei rack saturi
- [ ] Progettare la VPN e la sicurezza accessi (ISO 27001)
- [ ] Proporre un piano per Pondus (scalabilità, ridondanza)
- [ ] Dimensionare i nuovi 8 nodi HPC rispettando le specifiche tecniche
- [ ] Compilare il documento di offerta con tutte le sezioni richieste
- [ ] (Bonus) Preparare Gantt e tabella RASCI# Progetto Infrastruttura IT – TP Group srl
**Unità Formativa:** Learning by project | **Docente:** Luca Robaldo

---

## Contesto

Due aziende del settore automotive/aerospace si fondono per creare **TP Group srl**:

- **Thema Consulting** – fondata fine anni '90, 30 persone, sede di proprietà a Torino, data center on-premise (2 rack saturi), cluster HPC, applicazione proprietaria *Pondus*
- **Pro Studio** – startup 2023, 20 persone, nessuna infrastruttura IT interna, tutto su cloud (Google Workspace, GCP, SaaS)

Con la fusione, **i dipendenti di Pro Studio si trasferiscono nella sede di Thema Consulting**. Il contratto di affitto di Pro Studio non viene rinnovato.

---

## Cosa dobbiamo fare

Redigere un **documento di offerta tecnico/economica** per la convergenza e il potenziamento dei sistemi IT delle due aziende. Il documento deve rispettare i requisiti elencati nel capitolato.

---

## Situazione attuale (da integrare)

| Area | Thema Consulting | Pro Studio |
|---|---|---|
| Virtualizzazione | VMware (2 host) + SAN NetApp 24TB | Nessuna |
| Backup | NAS QNAP 32TB on-premise | Google Cloud (GCP) |
| Firewall | Fortigate FG-60F | N/A |
| Cluster HPC | 8 nodi fisici Linux + Infiniband | N/A |
| Email/Cal | Zimbra (VM Linux) | Google Workspace |
| Produttività | MS365 / LibreOffice | Google Workspace |
| Video/IM | MS Teams | Google Meet |
| Autenticazione | Active Directory (Samba4, VM Linux) | Google Authenticator |
| File Server | NextCloud (VM Linux) | Google Drive |
| Dev/Versioning | Gitea | Bitbucket (SaaS) |
| Contabilità | Profis (on-premise, Windows) | Odoo (SaaS) |
| CRM | SugarCRM (VM Linux) | Odoo (SaaS) |
| Timesheet | Kimai (VM Linux) | Odoo (SaaS) |
| Documentazione | MediaWiki (VM Linux) | MediaWiki (GCP) |
| App proprietaria | **Pondus** (IIS + MSSQL, hw fisico) | N/A |

---

## Vincoli da rispettare

### Tecnologici (ISO/IEC 27001)
- **Business continuity**: ridondanza dei sistemi critici e delle connettività
- **Disaster recovery**: backup delocalizzati, servizi clienti sempre fruibili
- **Sicurezza accessi**: cifratura storage, protocolli sicuri, VPN (LAN-to-LAN, Client-to-LAN, Hybrid cloud)

### Logistici
- Tutti i 50 dipendenti (30 + 20) lavoreranno nella sede Thema a Torino
- Il data center esistente **non può essere espanso** (rack saturi, condizionamento al limite)
- Ogni dipendente ha un laptop aziendale; **non si usano soluzioni VDI**

---

## Obiettivi futuri da includere nell'offerta

### Cluster HPC
- Aggiungere **8 nuovi nodi** entro 2 anni (totale 16)
- Nodi fisici (no VM, no container) – uso esclusivo personale interno
- Rete doppia: Ethernet + **Infiniband 100/200 Gbps**
- Specifiche per nodo: 2 CPU, ≥24 core, clock elevato, 8 GB RAM/core, no GPU, Linux, 1,6 TB scratch
- Budget: **250.000€ + IVA** (on-premise/hosting) oppure **220.000€ + IVA** (cloud/noleggio)

### Applicazione Pondus
- Potenziamento e distribuzione a un'utenza più ampia

---

## Struttura del documento da consegnare

Il documento deve contenere **obbligatoriamente**:

1. **Pagina introduttiva**
2. **Indice**
3. **Presentazione del team** (chi siete, servizi offerti, storia)
4. **Introduzione** – sintesi dell'offerta
5. **Assunzioni** – prerequisiti per l'erogazione del progetto
6. **Descrizione fornitura hardware** – materiali, esclusioni, note
7. **Attività richieste** – installazione HW/SW, test funzionali, esclusioni, note
8. **Documentazione** – cosa verrà rilasciato
9. **Termini e condizioni** (se presenti)
10. **Quotazione economica**

**Nice to have (bonus)**:
- Gantt di progetto
- Tabella RASCI

---

## Criteri di valutazione

La soluzione proposta sarà valutata rispetto a queste dimensioni:

| Dimensione | Opzioni da valutare |
|---|---|
| Erogazione del servizio | On-premise / Cloud / Hosting co-location (min. 3 anni) |
| Tipo di server | Fisico / Virtuale / Container |
| Sistema operativo | Windows Server / Linux |
| Tipo di soluzione | Open source / Commerciale |
| Proprietà beni IT | A noleggio / Di proprietà |
| Gestione IT | Personale interno / Outsourcing |
| Pricing | Outsourcing / Personale interno |

---

## Costi di riferimento per il piano economico

- Tecnico sistemista (≥5 anni exp): **500€/giorno**
- Project Manager: **650€/giorno**

---

## Checklist rapida

- [ ] Scegliere come unificare i servizi duplicati (email, file server, CRM, documentazione, ecc.)
- [ ] Decidere dove collocare i sistemi (on-premise, cloud, ibrido) tenendo conto dei rack saturi
- [ ] Progettare la VPN e la sicurezza accessi (ISO 27001)
- [ ] Proporre un piano per Pondus (scalabilità, ridondanza)
- [ ] Dimensionare i nuovi 8 nodi HPC rispettando le specifiche tecniche
- [ ] Compilare il documento di offerta con tutte le sezioni richieste
- [ ] (Bonus) Preparare Gantt e tabella RASCI