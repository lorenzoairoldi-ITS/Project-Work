# Vincoli — Fusione Thema + Pro Studio → TP Group

## Vincoli Logistici / Fisici

| Vincolo | Impatto |
|---------|---------|
| 2 rack saturi | Impossibile aggiungere HW on-premise senza rimuovere qualcosa — non c'è spazio per nuovi server |
| Condizionamento al limite | Non si può aumentare la densità di potenza nei rack → niente server più potenti o aggiuntivi |
| UPS 10 minuti | Autonomia insufficiente per uno spegnimento ordinato di tutti i sistemi |
| No VDI | Ogni dipendente ha un laptop aziendale — niente desktop virtuali |
| 50 persone nella stessa sede | Pro Studio perde sede (affitto non rinnovato), tutti si trasferiscono da Thema → carico raddoppiato su rete locale, firewall, server, condizionamento |
| Pro Studio non ha figura IT interna | Zero sistemisti: la gestione IT passa completamente a TP Group |

## Vincoli Tecnici / Infrastrutturali

| Vincolo | Impatto |
|---------|---------|
| Fortigate FG-60F sottodimensionato | Dimensionato per ≤30 utenti, con 50 utenti + VPN il throughput e le connessioni concorrenti sono insufficienti |
| Nessuna ridondanza firewall | Singolo FG-60F → se si guasta, l'intera connettività è persa |
| Backup Thema solo on-premise (QNAP) | Nessuna copia off-site → viola ISO/IEC 27001 per il DR. Il QNAP è consumer/prosumer, non enterprise |
| Pro Studio: zero backup indipendente | Dipendenza totale da Google: se account compromesso/bloccato, dati non accessibili |
| Pro Studio: zero sicurezza perimetrale | Nessun firewall, nessuna VPN, nessun IPS/IDS — i 20 utenti entreranno nella rete Thema senza protezioni |
| Pondus su HW fisico non ridondato | Singolo server IIS + MSSQL — single point of failure, difficile scalare, HW obsoleto |
| Samba4 AD non completo | Non implementa AD Federation Services, PowerShell remoting, GPO avanzate → limite per federazione con Google Identity |
| 40+ TB di dati cloud da migrare | 2 TB/utente × 20 utenti Pro Studio — volume significativo da trasferire |

## Vincoli HPC

| Vincolo | Dettaglio |
|---------|-----------|
| Nodi fisici obbligatori | No VM, no container — uso esclusivo personale interno |
| Rete Infiniband 100/200 Gbps | Obbligatoria per la bassa latenza richiesta da simulazioni |
| Specifiche tassative per nodo | 2 CPU, ≥24 core, clock elevato, 8 GB RAM/core, no GPU, 1,6 TB scratch SSD, Linux |
| Budget massimo | 250.000€ + IVA (on-prem/hosting) oppure 220.000€ + IVA (cloud/noleggio) |

## Vincoli Normativi (ISO/IEC 27001)

| Vincolo | Richiede |
|---------|----------|
| Business continuity | Ridondanza sistemi critici e connettività |
| Disaster recovery | Backup delocalizzati, RPO/RTO definiti, servizi sempre fruibili |
| Sicurezza accessi | Cifratura storage, protocolli sicuri, VPN (LAN-to-LAN, Client-to-LAN, Hybrid cloud) |
| Audit trail | Tracciabilità degli accessi e delle operazioni |

## Vincoli di Offerta / Consegna

| Vincolo | Dettaglio |
|---------|-----------|
| Documento con 10 sezioni obbligatorie | Introduttiva, indice, team, introduzione, assunzioni, hardware, attività, documentazione, termini, quotazione |
| Costi di riferimento | Sistemista senior: 500€/giorno — PM: 650€/giorno |
| 7 dimensioni di valutazione | Erogazione (on-prem/cloud/colo), tipo server (fisico/virtuale/container), OS (Windows/Linux), soluzione (open source/commerciale), proprietà (noleggio/proprietà), gestione (interno/outsourcing), pricing |

---

**Nota:** il vincolo più critico è lo spazio fisico esaurito nei rack di Thema — questa singola limitazione forza buona parte delle scelte architetturali verso cloud/colo o verso la rimozione di HW esistente per fare spazio ai nuovi nodi HPC.
