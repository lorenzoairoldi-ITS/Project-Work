# Piano di Migrazione — Pro Studio → Sede Thema

**Data:** 18/06/2026
**Stato:** Bozza — da validare

---

## Contesto

A seguito della fusione in **TP Group srl**:
- **Thema Consulting:** 30 dipendenti, sede Torino (di proprietà), infrastruttura on-premise
- **Pro Studio:** 20 dipendenti, sede Torino (affitto — non rinnovato), modello cloud-native
- **Obiettivo:** tutti i 50 dipendenti lavoreranno nella sede Thema a Torino

---

## Vincoli

| Vincolo | Impatto |
|---------|---------|
| 2 rack saturi | Impossibile aggiungere HW on-premise |
| Condizionamento al limite | Impossibile aumentare densità di potenza |
| Fortigate FG-60F | Dimensionato per ≤30 utenti |
| Backup solo on-premise | Viola requisiti ISO 27001 (nessuna copia off-site) |
| Pondus su HW fisico | Nessuna ridondanza, difficile scalare |
| ~40TB dati Pro Studio su cloud | Volume significativo da migrare |
| Nessuna figura IT in Pro Studio | Gestione IT completamente demandata a TP Group |

---

## Architettura target: Cloud ibrido

Dato che i rack sono saturi e il condizionamento è al limite, si adotta un **modello ibrido**:

| Cosa | Dove | Motivo |
|------|------|--------|
| HPC cluster (8+8 nodi) | **On-premise** | Infiniband, latenza, dati volumetrici |
| Pondus | **VM su hosting esterno** | Rack saturi, necessità di HA e scaling |
| Servizi IT (Zimbra, NextCloud, ecc.) | **Hosting esterno / IaaS** | Rack saturi, possibilità di ridondanza |
| Backup DR | **Cloud / secondo sito** | ISO 27001 — copia off-site obbligatoria |

---

## Piano migrazione — servizio per servizio

### 1. Email / Calendar — Zimbra (unificato)

| | Thema | Pro Studio |
|---|---|---|
| Attuale | Zimbra (VM Linux) | Gmail / Google Calendar |
| Target | **Zimbra potenziato** | |

**Azioni:**
- Ridimensionare VM Zimbra per 50 utenti
- Migrare mailbox Pro Studio via imapsync
- Migrare calendari e contatti (esportazione ICS + import)
- Mantenere forward da dominio Pro Studio verso nuovo dominio unificato

**Complessità:** Media

---

### 2. Produttività — MS365 (unificato)

| | Thema | Pro Studio |
|---|---|---|
| Attuale | MS365 + LibreOffice | Google Docs / Sheets / Slides |
| Target | **MS365** | |

**Azioni:**
- Acquistare licenze MS365 per i 20 utenti Pro Studio
- Esportare documenti da Google Docs/Sheets/Slides in formato Office
- Formazione utenti Pro Studio su Office e Teams

**Complessità:** Bassa

---

### 3. Video / IM — Teams (unificato)

| | Thema | Pro Studio |
|---|---|---|
| Attuale | MS Teams | Google Meet / Chat |
| Target | **MS Teams** | |

**Azioni:**
- Creare canali e team per i nuovi utenti
- Disattivare Google Meet/Chat per TP Group
- Formazione su Teams per utenti Pro Studio

**Complessità:** Bassa

---

### 4. AD / Identità — Samba4 AD esteso + federazione

| | Thema | Pro Studio |
|---|---|---|
| Attuale | Samba4 AD (VM Linux) | Google Identity (2FA/SSO) |
| Target | **Samba4 AD esteso** con federazione SAML | |

**Azioni:**
- Creare account AD per i 20 utenti Pro Studio
- Configurare federazione SAML tra Google Identity e Samba4 AD (transition)
- Unificare le policy di accesso (password, 2FA)
- Disattivare gradualmente Google Identity come IdP primario

**Complessità:** Alta

---

### 5. File Server — NextCloud (unificato)

| | Thema | Pro Studio |
|---|---|---|
| Attuale | NextCloud (VM Linux) | Google Drive (~2TB/utente, ~40TB totali) |
| Target | **NextCloud potenziato** | |

**Azioni:**
- Ridimensionare storage NextCloud per gestire ~40TB aggiuntivi
- Pianificare migrazione dati a lotti per utente
- Utilizzare strumenti di sync (rclone / Google Drive API → NextCloud API)
- Riorganizzare permessi e strutture di condivisione
- Formazione utenti su interfaccia NextCloud
- Mantenere accesso a Google Drive in sola lettura per il periodo di transizione

**Complessità:** Molto alta

---

### 6. Dev / Versioning — Gitea (unificato)

| | Thema | Pro Studio |
|---|---|---|
| Attuale | Gitea (VM Linux) | Bitbucket (SaaS Atlassian) |
| Target | **Gitea** | |

**Azioni:**
- Clonare tutti i repository da Bitbucket (git clone --mirror)
- Pushare su Gitea (git push --mirror)
- Verificare integrità repository e permessi
- Disattivare account Bitbucket Pro Studio

**Complessità:** Bassa

---

### 7. CRM — Odoo (unificato)

| | Thema | Pro Studio |
|---|---|---|
| Attuale | SugarCRM (VM Linux) | Odoo CRM (SaaS) |
| Target | **Odoo** | |

**Azioni:**
- Esportare dati da SugarCRM (contatti, opportunità, contratti, storico)
- Mappare campi tra modello SugarCRM e Odoo CRM
- Importare dati in Odoo
- Verificare integrità dello storico commerciale
- Spegnere VM SugarCRM dopo verifica

**Complessità:** Alta

---

### 8. Timesheet — Odoo (unificato)

| | Thema | Pro Studio |
|---|---|---|
| Attuale | Kimai (VM Linux) | Odoo Timesheet (SaaS) |
| Target | **Odoo Timesheet** (se si adotta Odoo come ERP) | |

**Azioni:**
- Esportare dati timesheet da Kimai (CSV/API)
- Importare in Odoo Timesheet
- Spegnere VM Kimai dopo verifica

**Complessità:** Media

---

### 9. Contabilità — Mantenimento separato (transizione)

| | Thema | Pro Studio |
|---|---|---|
| Attuale | Profis (on-premise, Windows) | Odoo Accounting (SaaS) |
| Target | **Mantenere entrambi** nel breve periodo | |

**Azioni:**
- Mantenere Profis e Odoo Accounting in parallelo per il primo periodo post-fusione
- Valutare migrazione graduale su ERP unico (Odoo come candidato)
- Coinvolgere commercialista per mappatura piani dei conti

**Complessità:** Molto alta

---

### 10. Documentazione — MediaWiki (unificato)

| | Thema | Pro Studio |
|---|---|---|
| Attuale | MediaWiki (VM Linux) | MediaWiki (VM GCP) |
| Target | **MediaWiki Thema** (istanza unica) | |

**Azioni:**
- Esportare pagine da MediaWiki Pro Studio (XML)
- Importare su MediaWiki Thema
- Riconciliare categorie, template, namespace
- Redirect o spegnimento VM GCP

**Complessità:** Bassa

---

### 11. Pondus — Virtualizzazione e ridondanza

| Attuale | Target |
|---|---|
| IIS + MSSQL su HW fisico dedicato | **VM su hosting esterno con ridondanza** |

**Azioni:**
- Migrare Pondus da HW fisico a VM (P2V con VMware vCenter Converter o reinstall)
- Configurare cluster MSSQL (always-on) per HA
- Bilanciamento carico IIS (Web Farm)
- Test di carico con 50+ utenti
- Piano di disaster recovery per Pondus

**Complessità:** Alta

---

## Ordine cronologico delle attività

### Fase 0 — Propedeutiche
- [ ] Sostituzione Fortigate FG-60F con modello per 50+ utenti (es. FG-100F) con HA
- [ ] Attivazione contratto hosting / IaaS per servizi da delocalizzare
- [ ] Attivazione backup off-site (NAS secondario o cloud)
- [ ] Dimensionamento storage NextCloud per ~40TB aggiuntivi

### Fase 1 — Bassa complessità
- [ ] Unificazione MediaWiki (esportazione XML + import)
- [ ] Migrazione repository Bitbucket → Gitea
- [ ] Configurazione Teams per nuovi utenti

### Fase 2 — Media complessità
- [ ] Migrazione email Gmail → Zimbra (imapsync)
- [ ] Estensione Samba4 AD ai 20 utenti Pro Studio
- [ ] Acquisizione licenze MS365 e migrazione documenti

### Fase 3 — Alta complessità
- [ ] Migrazione Google Drive → NextCloud (~40TB, a lotti per utente)
- [ ] Migrazione SugarCRM → Odoo (mappatura campi)
- [ ] Virtualizzazione Pondus (P2V) e configurazione ridondanza
- [ ] Migrazione timesheet Kimai → Odoo

### Fase 4 — Lunghissimo termine
- [ ] Valutazione unificazione contabile Profis → Odoo

---

## Gestione del cambiamento

| Attività | Target |
|---|---|
| Formazione utenti Pro Studio su Zimbra | 1 sessione per utente (2h) |
| Formazione utenti Pro Studio su NextCloud | 1 sessione per utente (1h) |
| Formazione utenti Pro Studio su Teams | 1 sessione gruppale (1h) |
| Formazione utenti Pro Studio su MS365 | 1 sessione per utente (2h) |
| Formazione utenti Thema su Odoo (CRM/Timesheet) | 1 sessione gruppale (2h) |
| Formazione utenti Pro Studio su Odoo (se diverso) | 1 sessione per utente (1h) |

---

## Rischi principali

| Rischio | Probabilità | Impatto | Mitigazione |
|---------|:-----------:|:-------:|-------------|
| Dimensione migrazione file (~40TB) | Alta | Alto | Migrazione a lotti, finestre notturne, bandwidth limit |
| Resistenza utenti Pro Studio al cambio strumenti | Media | Medio | Formazione adeguata, periodo di transizione |
| Complessità migrazione contabile | Alta | Alto | Mantenere separato, migrazione graduale |
| Perdita dati CRM durante mappatura campi | Media | Alto | Backup completo SugarCRM pre-migrazione, verifica post |
| Rack saturi bloccano upgrade imprevisti | Alta | Alto | Tutto il possibile va su hosting esterno |

---

*Documento basato sulle analisi AS-IS di Thema Consulting e Pro Studio.*
