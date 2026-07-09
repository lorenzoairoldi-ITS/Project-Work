# Analisi AS-IS — Pro Studio

**Data:** 18/06/2026\
**Redatto da:** 5Stack\
**Versione:** 1.0

---

## 1. Profilo aziendale

| Dato | Valore |
|------|--------|
| Nome | Pro Studio |
| Fondazione | 2023 |
| Settore | Automotive / Aerospace |
| Dipendenti | 20 |
| Sede | Torino (in affitto — contratto non rinnovato) |
| Data center | Nessuno — tutto su cloud |
| Modello IT | Cloud-native (Google Workspace, GCP, SaaS) |

---

## 2. Infrastruttura virtualizzazione

### 2.1 Nessuna — cloud-native

**Cosa usano:** Nessun hypervisor, nessun server fisico, nessuna SAN. L'intera infrastruttura IT è affidata a servizi cloud SaaS e IaaS.

**Perché cloud-native:**
- Zero investimento iniziale in hardware — tipico di una startup con capitale limitato
- Scalabilità elastica: paghi per quello che usi
- Manutenzione zero: Google si occupa di aggiornamenti, patch, uptime
- Time-to-market rapido: si attiva un servizio in pochi minuti, senza attendere forniture hardware

**Contesto:**
Questa scelta ha permesso a Pro Studio di operare da subito con 20 persone senza assumere un tecnico sistemista e senza acquistare server. L'intera gestione IT è demandata ai fornitori cloud. Con la fusione in TP Group, questo modello va riconciliato con l'infrastruttura on-premise di Thema.

---

## 3. Backup

### 3.1 Google Cloud (GCP)

**Cosa usano:** Backup automatici dei dati tramite i meccanismi nativi di Google Workspace e GCP.

**Perché GCP:**
- Backup inclusi nel costo del servizio (nessun investimento aggiuntivo)
- Ridondanza geografica dei dati garantita da Google (repliche multi-regione)
- Versioning dei file su Google Drive (cronologia versioni, cestino)

**Criticità:**
- Nessuna copia di backup indipendente da Google: se l'account viene compromesso o bloccato, i dati non sono accessibili
- Dipendenza totale dal provider: eventuali interruzioni di GCP bloccano l'accesso a tutti i dati
- Assenza di una strategia di backup cross-provider o backup locale
- Rischio di lock-in: migrare 20 utenti da Google Workspace a un altro sistema richiede pianificazione

---

## 4. Firewall e rete

### 4.1 N/A — nessuna infrastruttura di rete fisica

**Cosa usano:** Nessun firewall aziendale, nessuna VPN, nessuna segmentazione di rete. Tutti i servizi sono erogati via Internet tramite browser web.

**Perché:**
- I dipendenti lavorano da remoto o in sede con laptop connessi direttamente a Internet
- Google Workspace e i SaaS usati (Odoo, Bitbucket) sono accessibili via HTTPS
- Nessuna risorsa on-premise da proteggere

**Criticità:**
- **Zero sicurezza perimetrale:** nessun firewall, nessun IPS/IDS, nessun filtro web
- **Nessuna VPN aziendale:** il traffico dei dipendenti non è instradato attraverso un gateway centrale
- **Controllo accessi solo via Google:** la sicurezza è affidata unicamente a Google Workspace (2FA, SSO)
- Con la fusione, i dipendenti Pro Studio si connetteranno alla rete di Thema: servirà una VPN client-to-LAN per accedere alle risorse on-premise (Pondus, HPC, file server)
- Il firewall Fortigate FG-60F di Thema (già saturo con 30 utenti) dovrà gestire anche il traffico dei 20 nuovi utenti

---

## 5. Cluster HPC

### 5.1 N/A — nessun cluster

**Cosa usano:** Nessun cluster HPC. I dipendenti Pro Studio non eseguono simulazioni o calcolo ad alte prestazioni.

**Criticità:**
- Nessuna criticità diretta. Tuttavia, in ottica TP Group, i dipendenti Pro Studio potrebbero aver bisogno di accedere al cluster HPC di Thema per progetti condivisi.
- Accesso ai nodi HPC da gestire tramite VPN e autenticazione centralizzata.

---

## 6. Servizi IT

### 6.1 Email / Calendario — Google Workspace (Gmail, Calendar)

**Cosa usano:** Gmail per email aziendale e Google Calendar per appuntamenti e calendari condivisi.

**Perché Google Workspace:**
- Soluzione SaaS matura, nessuna gestione server
- Ottima esperienza utente, app mobili native
- Collaborazione in tempo reale, ricerca potente
- Costo prevedibile (abbonamento per utente/mese)

**Criticità:**
- Thema usa Zimbra (self-hosted): i due sistemi vanno unificati
- La migrazione da Google Workspace a Zimbra (o a una terza piattaforma) comporta:
  - Migrazione delle mailbox (strumenti come imapsync)
  - Migrazione dei calendari e contatti
  - Formazione utenti sul nuovo client
- In alternativa, si potrebbe migrare Thema su Google Workspace — valutazione economica e organizzativa necessaria

### 6.2 Produttività — Google Workspace (Docs, Sheets, Slides)

**Cosa usano:** Google Documenti, Fogli e Presentazioni per la produttività individuale e collaborativa.

**Perché Google Workspace:**
- Editing collaborativo in tempo reale nativo (senza necessità di salvataggi o condivisione file)
- Storico versioni automatico
- Accesso da qualsiasi dispositivo
- Integrazione nativa con Gmail e Google Drive

**Criticità:**
- Thema usa MS365 + LibreOffice: due approcci completamente diversi
- I formati Google non sono pienamente compatibili con Office (problemi di formattazione in .docx/.xlsx)
- Bisogna scegliere una suite standard per TP Group:
  - **Opzione A:** migrare Pro Studio su MS365 (costo licenze + formazione)
  - **Opzione B:** migrare Thema su Google Workspace (cambio abitudini 30 utenti)
  - **Opzione C:** mantenere due piattaforme (doppia gestione, sconsigliato)

### 6.3 Video / IM — Google Meet / Google Chat

**Cosa usano:** Google Meet per videoconferenze e Google Chat per messaggistica istantanea.

**Perché Google Meet/Chat:**
- Integrato con Google Workspace
- Meet: fino a 100 partecipanti, registrazione, sottotitoli automatici
- Chat: messaggi diretti, stanze, integrazione con Drive e Calendar

**Criticità:**
- Thema usa MS Teams: scelta obbligata su una piattaforma unica
- Teams ha più funzionalità enterprise (canali, tab, app, integrazione SharePoint)
- Migrazione dei meeting esistenti e delle chat

### 6.4 Autenticazione — Google Authenticator / Google Identity

**Cosa usano:** Google Workspace Identity come provider di identità (IdP) con 2FA via Google Authenticator.

**Perché Google Identity:**
- Integrato con Google Workspace, nessun costo aggiuntivo
- SSO per tutti i servizi Google (Gmail, Drive, Meet)
- 2FA via app Authenticator o SMS
- Gestione utenti centralizzata nella console Google Admin

**Criticità:**
- Thema usa Samba4 AD: i due domini di identità vanno unificati
- Scenari possibili:
  - **Estendere Samba4 AD** ai 20 utenti Pro Studio (migrazione identità)
  - **Passare a un IdP cloud** (Azure AD / Google Identity) come ponte tra i due mondi
  - **Federazione** tra Google Identity e Samba4 AD via SAML/LDAP
- La gestione separata degli account (Google + AD) è insostenibile per TP Group

### 6.5 File Server — Google Drive

**Cosa usano:** Google Drive (shared drives) per archiviazione e condivisione file.

**Perché Google Drive:**
- Sincronizzazione automatica su tutti i dispositivi
- Condivisione con permessi granulari (viewer/editor/commentator)
- Cestino e cronologia versioni
- 2 TB/utente (tipico piano Business)

**Criticità:**
- Thema usa NextCloud self-hosted: due sistemi di file serving
- I 20 utenti Pro Studio hanno circa 40 TB di dati da migrare (stima 2 TB/utente)
- La migrazione comporta:
  - Trasferimento dati su NextCloud via rete (dimensione banda disponibile)
  - Riorganizzazione delle strutture di condivisione e permessi
  - Formazione utenti sull'interfaccia NextCloud
- In alternativa: mantenere Google Drive e affiancarlo a NextCloud (sconsigliato per complessità)

### 6.6 Dev / Versioning — Bitbucket (SaaS Atlassian)

**Cosa usano:** Bitbucket Cloud di Atlassian per repository Git e code review.

**Perché Bitbucket:**
- SaaS, nessuna gestione server
- Integrazione con Jira (se usano Atlassian per project management)
- Pull request, code review, pipeline CI/CD integrate
- Pricing per utente/mese

**Criticità:**
- Thema usa Gitea self-hosted: due piattaforme di versioning
- Il codice dei progetti automotive/aerospace potrebbe richiedere hosting on-premise per ragioni di sicurezza (IP sensibile)
- Scenari:
  - **Unificare su Gitea:** migrare repo da Bitbucket a Gitea (git remote update)
  - **Unificare su Bitbucket:** se il SaaS è accettabile anche per Thema
  - **Mantenere entrambi:** possibile se i team hanno esigenze separate (ma sconsigliato)

### 6.7 Contabilità — Odoo (SaaS)

**Cosa usano:** Modulo contabilità di Odoo in cloud (SaaS).

**Perché Odoo:**
- Suite ERP completa con moduli integrati (contabilità, CRM, timesheet, magazzino, fatturazione)
- SaaS: nessuna installazione o manutenzione
- Aggiornamenti automatici
- Accesso da browser

**Criticità:**
- Thema usa Profis (on-premise, Windows): due sistemi contabili
- Odoo e Profis hanno logiche contabili diverse (piano dei conti, IVA, registrazioni)
- La migrazione/unificazione dei dati contabili è complessa e richiede un commercialista
- Output consigliato: mantenere entrambi i sistemi per il primo periodo post-fusione, pianificare una migrazione graduale su un unico ERP

### 6.8 CRM — Odoo (SaaS)

**Cosa usano:** Modulo CRM di Odoo in cloud.

**Perché Odoo CRM:**
- Integrato con la contabilità e il resto della suite Odoo
- Gestione lead, opportunità, attività, preventivi
- SaaS: nessuna gestione server

**Criticità:**
- Thema usa SugarCRM self-hosted: due CRM da consolidare
- SugarCRM e Odoo CRM hanno modelli dati diversi: la migrazione richiede mappatura campi e storico
- Scelta obbligata: unificare su una piattaforma unica (Odoo o SugarCRM o una terza)
- Lo storico commerciale (opportunità, contratti, comunicazioni) va preservato

### 6.9 Timesheet — Odoo (SaaS)

**Cosa usano:** Modulo timesheet di Odoo per la registrazione delle ore lavorate.

**Perché Odoo Timesheet:**
- Integrato con il CRM e la contabilità di Odoo
- Tracciamento ore per progetto/commessa
- Dashboard e reportistica integrata

**Criticità:**
- Thema usa Kimai self-hosted: due sistemi di timesheet
- Kimai è molto più leggero e semplice di Odoo Timesheet
- Scelta: unificare su Odoo Timesheet (se si adotta Odoo come ERP unico) o mantenere Kimai
- I dati timesheet di Pro Studio vanno esportati da Odoo e importati nel sistema target

### 6.10 Documentazione — MediaWiki (GCP)

**Cosa usano:** MediaWiki ospitato su una VM Google Cloud Platform.

**Perché MediaWiki:**
- Standard de facto per documentazione tecnica (Wikipedia engine)
- Stessa identica piattaforma usata da Thema Consulting (ma on-premise, non su GCP)
- Categorie, template, allegati, cronologia

**Criticità:**
- Due istanze separate di MediaWiki: una su GCP (Pro Studio), una on-premise (Thema)
- Unificazione semplice perché stesso software:
  - Esportare pagine da MediaWiki Pro Studio (XML)
  - Importare su MediaWiki Thema
  - Riconciliare categorie e template
  - Redirect o spegnimento dell'istanza GCP
- **Nessuna criticità tecnica maggiore**: è il servizio più facile da unificare

---

## 7. Applicazione proprietaria

### 7.1 N/A — nessuna

**Cosa usano:** Nessuna applicazione sviluppata internamente.

**Criticità:**
- Nessuna criticità diretta. Tuttavia, i dipendenti Pro Studio potrebbero dover utilizzare **Pondus** (l'app proprietaria di Thema) dopo la fusione.
- Pondus è attualmente su IIS + MSSQL (HW fisico, non ridondato): va potenziato per supportare 20 utenti aggiuntivi.

---

## 8. Obiettivi strategici di Pro Studio

| Obiettivo | Dettaglio |
|-----------|-----------|
| **Transizione senza impatto** | I 20 dipendenti devono continuare a lavorare senza interruzioni durante e dopo la fusione |
| **Unificazione servizi cloud** | Convergenza di Google Workspace, Odoo e Bitbucket con l'infrastruttura on-premise di Thema |
| **Migrazione dati** | Trasferimento ordinato di email, file, CRM, timesheet e documentazione |
| **Minimo change management** | Formazione utenti mirata, interfacce familiari dove possibile |
| **Continuità operativa** | Nessun downtime dei servizi durante la migrazione |

---

## 9. Vincoli fisici

| Vincolo | Impatto |
|---------|---------|
| **Nessuna sede propria** | I dipendenti si trasferiscono nella sede Thema; niente infrastruttura fisica da smantellare |
| **Nessun HW on-premise** | Zero asset hardware da dismettere o riconvertire |
| **40+ TB di dati cloud** | Volume di dati da migrare su NextCloud / infrastruttura TP Group |
| **Nessuna VPN/rete aziendale** | Da progettare l'accesso dei nuovi utenti alla rete di Thema |
| **Nessuna figura IT interna** | Pro Studio non ha sistemisti: la gestione IT è completamente demandata a TP Group post-fusione |

---

## 10. Schema riassuntivo infrastruttura

```
┌─────────────────────────────────────────────────────────────┐
│                      PRO STUDIO                              │
│                        20 utenti                             │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────────────────────────────────────────┐    │
│  │                 GOOGLE WORKSPACE                      │    │
│  │                                                       │    │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌────────┐  │    │
│  │  │  Gmail  │  │  Drive  │  │  Docs/  │  │  Meet  │  │    │
│  │  │ Calendar│  │  (2TB/  │  │  Sheets │  │  Chat  │  │    │
│  │  │         │  │  utente)│  │  Slides │  │        │  │    │
│  │  └─────────┘  └─────────┘  └─────────┘  └────────┘  │    │
│  │                                                       │    │
│  │         ┌──────────────────┐  ┌──────────────────┐    │    │
│  │         │   Google Auth    │  │  MediaWiki GCP   │    │    │
│  │         │   (2FA / SSO)    │  │  (VM MediaWiki)  │    │    │
│  │         └──────────────────┘  └──────────────────┘    │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐    │
│  │                     SAAS                             │    │
│  │                                                       │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐           │    │
│  │  │  Odoo    │  │  Odoo    │  │  Odoo    │           │    │
│  │  │Contabilità│  │   CRM    │  │Timesheet │           │    │
│  │  └──────────┘  └──────────┘  └──────────┘           │    │
│  │                                                       │    │
│  │  ┌──────────┐                                        │    │
│  │  │ Bitbucket│                                        │    │
│  │  │  (Git)   │                                        │    │
│  │  └──────────┘                                        │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌───────────────────────────────────────────────┐           │
│  │       CONNESSIONE CON THEMA (POST-FUSIONE)     │           │
│  │                                                │           │
│  │  20 utenti → VPN → Rete Thema → Firewall       │           │
│  │                              Fortigate FG-60F   │           │
│  └───────────────────────────────────────────────┘           │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

*Fine analisi AS-IS Pro Studio — 18/06/2026*
