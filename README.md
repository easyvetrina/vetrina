# 🎫 Carta Fedeltà 2.0 — White-Label Loyalty Platform

> Piattaforma white-label per carta fedeltà digitale, rivendibile a qualsiasi attività (barbieri, parrucchieri, ristoranti, negozi, palestre, ecc.). Integrata con [Prenoty](https://prenoty.it) per gli abbonati.

---

## 📋 Panoramica

Un **unico sito web** (single codebase, single deploy) che serve infiniti negozi. Ogni negozio ha la sua URL e il sito si configura dinamicamente leggendo i dati dal database PostgreSQL su DigitalOcean.

### Due modalità di funzionamento

| | **Modalità A — Prenoty** | **Modalità B — Standalone** |
|---|---|---|
| **Costo** | Gratis (incluso nell'abbonamento Prenoty) | A pagamento (piani Base/Pro) |
| **Dati** | Letti dal DB Prenoty esistente | Creati ex novo dall'admin |
| **Clienti** | Già registrati, login diretto | Registrazione da zero |
| **Pannello Admin** | Completo (fedeltà + prenotazioni + calendario) | Solo funzionalità carta fedeltà |
| **Passaggio** | — | Upgrade fluido a Prenoty in qualsiasi momento |

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
- Bottone **"Prenota Ora"** (solo negozi Prenoty)
- Multi-lingua IT/EN (estensibile)

### Area Cliente
- **QR Code personale** per timbratura rapida al banco
- Carta timbri digitale con animazioni
- **Livelli fedeltà** Bronze 🥉 / Silver 🥈 / Gold 🥇
- Storico premi con livello associato
- Form per lasciare recensione (post-premio)
- Google Wallet integration
- Notifiche push via Firebase FCM

### Pannello Admin
- **Dashboard Analytics** — KPI animate, grafici timbri settimanali, donut stato carte, clienti mensili, distribuzione livelli
- **Gestione Clienti** — tabella con ricerca, livello, timbra/premio con popup conferma, confetti al completamento
- **QR Scanner** — scansione QR del cliente per timbratura istantanea
- **Gestione Recensioni** — approvazione, rifiuto, eliminazione
- **Notifiche Push** — invio manuale + toggle notifiche automatiche (benvenuto, nuovo timbro, premio, promemoria inattività)
- **Impostazioni complete** — info attività, colori brand live, font, carta fedeltà (timbri, premi), livelli con premi personalizzabili, password admin
- **Sezione Prenoty** (solo abbonati) — calendario, prenotazioni, statistiche

### Personalizzazione Autonoma
Ogni negoziante dal pannello admin personalizza in autonomia:
- Logo e foto del negozio
- Colori brand (primario, accento)
- Nome, tipo e descrizione attività
- Servizi con prezzi
- Operatori / team
- Orari di apertura
- Contatti (indirizzo, telefono, email)
- Impostazioni carta fedeltà
- Livelli e premi per livello

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
| **Frontend** | HTML + CSS + JavaScript puro (single-file) |
| **Backend** | Da definire (API REST per comunicare con DB) |
| **Database** | PostgreSQL su DigitalOcean (condiviso con Prenoty) |
| **Hosting** | DigitalOcean (App Platform / Droplet) |
| **Notifiche** | Firebase Cloud Messaging (FCM) — gratuito |
| **Wallet** | Google Wallet API — gratuito |
| **Pagamenti** | Stripe |
| **App mobile Prenoty** | Flutter / Dart |

---

## 📁 Struttura Progetto

```
carta-fedelta-2.0/
├── index.html              # Frontend completo (single-file, prototipo attuale)
├── README.md               # Questo file
├── TODO.md                 # Piano di sviluppo dettagliato
├── api/                    # Backend API (da implementare)
│   ├── server.js           # Entry point
│   ├── routes/
│   │   ├── auth.js         # Login, registrazione, sessioni
│   │   ├── shops.js        # CRUD negozi, config, personalizzazione
│   │   ├── clients.js      # CRUD clienti, timbri, premi
│   │   ├── reviews.js      # CRUD recensioni
│   │   ├── notifications.js # FCM push notifications
│   │   ├── wallet.js       # Google Wallet pass
│   │   ├── stripe.js       # Billing e abbonamenti
│   │   └── superadmin.js   # Gestione piattaforma
│   ├── middleware/
│   │   ├── auth.js         # JWT / session auth
│   │   └── prenoty.js      # Detect negozio Prenoty
│   ├── db/
│   │   ├── schema.sql      # Schema PostgreSQL
│   │   └── migrations/     # Migrazioni DB
│   └── config/
│       └── firebase.js     # Config FCM
├── public/                 # Assets statici
│   └── uploads/            # Loghi, foto negozi
└── deploy/
    └── digitalocean.yml    # Config deploy
```

---

## 🚀 Getting Started

### Prerequisiti
- Node.js >= 18
- PostgreSQL >= 15
- Account DigitalOcean
- Account Firebase (per FCM)
- Account Stripe (per pagamenti)
- Account Google Cloud (per Wallet API)

### Setup locale

```bash
# Clone
git clone https://github.com/tuouser/carta-fedelta-2.0.git
cd carta-fedelta-2.0

# Installa dipendenze API
cd api && npm install

# Configura variabili ambiente
cp .env.example .env
# Edita .env con le tue credenziali

# Crea schema DB
psql -U postgres -d cartafedelta -f api/db/schema.sql

# Avvia server dev
npm run dev
```

### Variabili Ambiente (.env)

```env
# Database
DATABASE_URL=postgresql://user:pass@host:5432/cartafedelta

# Firebase
FIREBASE_PROJECT_ID=your-project
FIREBASE_PRIVATE_KEY=...
FIREBASE_CLIENT_EMAIL=...

# Stripe
STRIPE_SECRET_KEY=sk_...
STRIPE_WEBHOOK_SECRET=whsec_...

# Google Wallet
GOOGLE_WALLET_ISSUER_ID=...
GOOGLE_WALLET_KEY_FILE=...

# JWT
JWT_SECRET=your-secret-key

# Prenoty DB (per integrazione)
PRENOTY_DATABASE_URL=postgresql://user:pass@host:5432/prenoty
```

---

## 🔐 Accessi Demo (prototipo attuale)

| Ruolo | Credenziali |
|---|---|
| **Cliente** | Telefono: `+39 333 1111111` (o qualsiasi demo) |
| **Admin** | Password: `admin123` |
| **Super Admin** | Password: `super123` oppure telefono: `super` |

---

## 📄 Licenza

Proprietario — © 2026 Benedetto / Prenoty
