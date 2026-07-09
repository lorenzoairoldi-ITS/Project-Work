# Domande chiave per la fusione Thema + Pro Studio → TP Group

## Infrastruttura & Datacenter
1. I due rack di Thema sono saturi e il raffreddamento è al limite: come gestiamo l'aggiunta di 8 nodi HPC e le apparecchiature per i servizi di Pro Studio?
2. L'UPS di Thema regge solo 10 minuti: è sufficiente per uno spegnimento ordinato di tutti i servizi (inclusi quelli migrati)?
3. Pro Studio è full-cloud: ha senso mantenere un datacenter on-premise o si valuta un cloud ibrido/solo cloud?

## Rete & Sicurezza
4. Il Fortigate FG-60F è sottodimensionato per 50 utenti: quale firewall lo sostituisce e come segmentiamo la rete (HPC, ufficio, DMZ)?
5. Come realizziamo la VPN LAN-to-LAN tra i due siti (se Pro Studio ha sede remota) e un eventuale hybrid cloud?
6. Pro Studio non ha firewall né backup indipendente: come li mettiamo in compliance ISO 27001?

## Identità & Accessi
7. Samba4 AD (Thema) vs Google Identity (Pro Studio): federazione SAML/LDAP o migrazione completa a un unico Identity Provider?
8. Come gestiamo l'autenticazione multi-fattore per tutti i servizi unificati?

## Servizi & Collaborazione
9. Email: Zimbra (Thema) vs Gmail (Pro Studio) — migriamo tutti su una piattaforma o lasciamo convivere?
10. File server: NextCloud (Thema) vs Google Drive (Pro Studio) — si unifica? Dove finiscono i ~40+ TB di cloud data?
11. Productivity: LibreOffice (Thema) vs Google Docs (Pro Studio) vs MS365 — quale standard adottiamo?

## Applicazioni Core
12. Pondus (C# / IIS / MSSQL) è un single point of failure su HW fisico: come lo rendiamo altamente disponibile e lo eventualmente containerizziamo?
13. Gitea (Thema) vs Bitbucket (Pro Studio): quale piattaforma Git unificata?
14. Odoo (Pro Studio) vs Profis + SugarCRM (Thema): si unificano ERP e CRM?

## HPC
15. L'espansione HPC (8 nodi, 220-250k EUR) come si integra con la rete Infiniband esistente?
16. Il nuovo cluster deve essere accessibile anche agli utenti Pro Studio? Con che policy di accesso?

## Backup & Disaster Recovery
17. Thema ha backup solo on-premise (QNAP), Pro Studio ha solo backup nativo Google: come progettiamo un piano di backup off-site conforme ISO 27001?
18. Qual è l'RPO e l'RTO target per ogni servizio (email, ERP, Pondus, file server, HPC)?

## Governance & Costi
19. Chi gestisce l'IT dopo la fusione: un team unificato? Figure interne o in outsourcing?
20. Il modello di servizio sarà on-premise, cloud, o ibrido? Qual è il TCO a 3 anni?
21. Il passaggio va gestito con change management minimo (Pro Studio era cloud-native): come riduciamo l'impatto sugli utenti?
