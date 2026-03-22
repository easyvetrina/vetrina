# 🎫 Carta Fedeltà 2.0 — White-Label Loyalty Platform

> Piattaforma white-label per carta fedeltà digitale, rivendibile a qualsiasi attività (barbieri, parrucchieri, ristoranti, negozi, palestre, ecc.). Integrata con [Prenoty](https://prenoty.it) per gli abbonati.

---

## 📋 Panoramica

Un **unico sito web** (single codebase, single deploy) che serve infiniti negozi. Ogni negozio ha la sua URL e il sito si configura dinamicamente leggendo i dati dal database PostgreSQL su DigitalOcean.

### Due modalità di funzionamento

| | **Modalità A — Prenoty** | **Modalità B — Standalone** |
|---|---|---|
| **Costo** | Gratis (incluso nell'abbonamento Prenoty) | A pagamento (piani Base €19/Pro €29) |
| **Dati** | Letti dal DB Prenoty esistente | Creati ex novo dall'admin |
| **Clienti** | Già registrati, login diretto | Registrazione da zero |
| **Pannello Admin** | Completo (fedeltà + prenotazioni + calendario) | Solo funzionalità carta fedeltà |
| **Passaggio** | — | Upgrade fluido a Prenoty in qualsiasi momento |

---

## 🎨 Design Direction

**Stile: TailAdmin/Shadcn** — Clean, sharp, enterprise con tocchi premium.

- **Ispirazione**: TailAdmin, Shadcn UI, Linear, Vercel Dashboard
- **Tema**: Dark mode con gerarchia superfici (bg-0 → bg-5)
- **Font**: Geist (display + body) + Geist Mono (dati)
- **Colori**: Indigo (#6366F1) primario, palette accenti per KPI (green, yellow, blue)
- **Card**: Bordi sottili, hover con lift + shadow, transizioni smooth
- **Sidebar**: Sezioni organizzate, indicatore attivo con barra indigo, badge contatori
- **Tocchi premium dagli altri stili**: micro-animazioni e card con leggero blur (da Horizon), ombre morbide e angoli arrotondati (da Soft Gradient)
- **Animazioni**: Stagger entrata KPI, barre grafici animate, fade-up cambio pagina, micro-interazioni hover, confetti al completamento carta, shimmer skeleton loading, scan line QR scanner

**Perché questo stile**: Essendo white-label per attività diverse, il design neutro ma premium permette a ogni negoziante di personalizzare colori e logo senza che il risultato stoni. Scalabile quando si aggiungono funzionalità Prenoty.

---

## ✨ Funzionalità

### Sito Vetrina (pubblico)
- Hero section con CTA
- Chi Siamo
- Servizi e prezzi
- Team / Operatori
- Recensioni clienti (con approvazione admin)
- Orari di apertura (evidenzia giorno corrente)
- Contatti
- Bottone "Prenota Ora" (solo negozi Prenoty)
- Multi-lingua IT/EN (estensibile)

### Area Cliente
- QR Code personale per timbratura rapida al banco
- Carta timbri digitale con animazioni
- Livelli fedeltà Bronze / Silver / Gold
- Storico premi con livello associato
- Form per lasciare recensione (post-premio)
- Google Wallet integration
- Notifiche push via Firebase FCM

### Pannello Admin (✅ prototipo completato)
- Dashboard Analytics — 4 KPI animate con change %, grafici timbri settimanali, donut stato carte, clienti mensili, distribuzione livelli
- Gestione Clienti — tabella con barra timbri visuale, badge livello, ricerca, timbra/premio con popup conferma
- QR Scanner — viewport con scan line animata, corner markers, scansione istantanea
- Gestione Recensioni — filtri (tutte/in attesa/approvate), approvazione/eliminazione
- Notifiche Push — invio manuale + toggle notifiche automatiche
- Impostazioni complete — 5 sezioni: info attività, colori brand live, carta fedeltà, livelli, sicurezza
- Sezione Prenoty (solo abbonati) — calendario, prenotazioni, statistiche

### Personalizzazione Autonoma
Ogni negoziante personalizza in autonomia dal pannello admin:
- Logo e foto del negozio
- Colori brand (primario, accento) con color picker live
- Nome, tipo e descrizione attività
- Servizi con prezzi
- Operatori / team
- Orari di apertura
- Contatti
- Impostazioni carta fedeltà e livelli

### Super Admin Panel
- Metriche piattaforma globali (negozi, abbonati, MRR, utenti)
- Tabella tutti i negozi con stato abbonamento
- Gestione piani pricing (Base €19, Pro €29, Prenoty €49)

### Stripe Integration
- Checkout per negozi standalone
- Gestione abbonamenti e billing automatico

---

## 🏗️ Stack Tecnologico

| Layer | Tecnologia |
|---|---|
| **Frontend** | HTML + CSS + JavaScript puro |
| **Backend** | Da definire (API REST) |
| **Database** | PostgreSQL su DigitalOcean (condiviso con Prenoty) |
| **Hosting** | DigitalOcean |
| **Notifiche** | Firebase Cloud Messaging (FCM) — gratuito |
| **Wallet** | Google Wallet API — gratuito |
| **Pagamenti** | Stripe |
| **App mobile Prenoty** | Flutter / Dart |

---

## 📁 Struttura Progetto

```
carta-fedelta-2.0/
├── README.md                        # Questo file
├── TODO.md                          # Piano di sviluppo dettagliato (12 fasi)
├── prototypes/
│   ├── admin-panel-v2.html          # ✅ Pannello admin (stile Shadcn, funzionante)
│   └── carta-fedelta-2.0-full.html  # Prototipo completo v1 (da rifare con nuovo stile)
├── api/                             # Backend API (da implementare — Fase 1)
│   ├── server.js
│   ├── routes/
│   ├── middleware/
│   ├── db/
│   │   ├── schema.sql
│   │   └── migrations/
│   └── config/
├── public/
│   └── uploads/
└── deploy/
    └── digitalocean.yml
```

---

## 🔐 Accessi Demo (prototipi)

| Ruolo | Credenziali |
|---|---|
| **Cliente** | Telefono: `+39 333 111 1111` |
| **Admin** | Password: `admin123` |
| **Super Admin** | Password: `super123` |

---

## 📦 Stato Sviluppo

| Fase | Descrizione | Stato |
|---|---|---|
| 0 | Setup progetto e architettura | ⬜ Da fare |
| 1 | Infrastruttura base (API, DB, multi-tenant) | ⬜ Da fare |
| 2 | QR Code timbratura | ⬜ Da fare (prototipo FE pronto) |
| 3 | Sistema recensioni | ⬜ Da fare (prototipo FE pronto) |
| 4 | Multi-lingua | ⬜ Da fare |
| 5 | Livelli fedeltà | ⬜ Da fare (prototipo FE pronto) |
| 6 | Prenotazione rapida (Prenoty) | ⬜ Da fare |
| 7 | Super Admin Panel | ⬜ Da fare |
| 8 | Stripe / Pagamenti | ⬜ Da fare |
| 9 | Firebase FCM | ⬜ Da fare |
| 10 | Google Wallet | ⬜ Da fare |
| 11 | Deploy | ⬜ Da fare |
| 12 | Polish e ottimizzazioni | ⬜ Da fare |

### Prototipi Frontend completati:
- ✅ Admin Panel v2 (stile Shadcn — `prototypes/admin-panel-v2.html`)
- 🔄 Sito Vetrina (da rifare con stesso stile dell'admin)
- 🔄 Area Cliente (da rifare con stesso stile)
- 🔄 Super Admin Panel (da rifare con stesso stile)

---

## 📄 Licenza

Proprietario — © 2026 Benedetto / Prenoty
