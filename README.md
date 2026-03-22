# 🎫 Carta Fedeltà 2.0 — White-Label Loyalty Platform

> Piattaforma white-label per carta fedeltà digitale, rivendibile a qualsiasi attività (barbieri, parrucchieri, ristoranti, negozi, palestre, ecc.). Integrata con [Prenoty](https://prenoty.it) per gli abbonati.

---

## 📋 Panoramica

Un **unico sito web** (single codebase, single deploy) che serve infiniti negozi. Ogni negozio ha la sua URL e il sito si configura dinamicamente leggendo i dati dal database PostgreSQL su DigitalOcean. Nessun sito viene creato manualmente: il negoziante personalizza tutto dal suo pannello admin.

### Due modalità di funzionamento

| | **Modalità A — Prenoty** | **Modalità B — Standalone** |
|---|---|---|
| **Costo** | Gratis (incluso nell'abbonamento Prenoty) | A pagamento (piani Base €19/Pro €29) |
| **Dati negozio** | Letti dal DB Prenoty esistente (servizi, operatori, orari, contatti) | Inseriti manualmente dall'admin |
| **Clienti** | Già registrati nel DB Prenoty, login diretto senza re-registrazione | Registrazione da zero |
| **Pannello Admin** | Completo (fedeltà + prenotazioni + calendario + statistiche Prenoty) | Solo funzionalità carta fedeltà |
| **Passaggio** | — | Upgrade fluido a Prenoty in qualsiasi momento |

### Flusso multi-tenant

1. Il negozio accede alla propria URL (es. `cartafedelta.it/barbiere-mario`)
2. Il backend legge lo `slug` dall'URL → carica config dal DB PostgreSQL
3. Il sito si configura dinamicamente (logo, colori, servizi, dati)
4. Se `is_prenoty=true`: i dati vengono letti dal DB Prenoty (read-only), i clienti esistono già
5. Se `is_prenoty=false`: i dati sono gestiti localmente, i clienti si registrano da zero

### Upgrade Standalone → Prenoty

Quando un negozio standalone si abbona a Prenoty:
- Il flag `is_prenoty` viene impostato a `true` e viene collegato il `prenoty_shop_id`
- I dati negozio (servizi, operatori, orari) vengono sincronizzati dal DB Prenoty
- I clienti esistenti vengono **unificati per numero di telefono** con quelli già presenti in Prenoty
- Il pannello admin si aggiorna **live** mostrando le sezioni Prenoty (calendario, prenotazioni, statistiche)
- Nessuna perdita di dati: timbri, premi e livelli vengono mantenuti

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

### Sito Vetrina (pubblico) — ✅ prototipo completato
- Hero section con CTA e stats animate
- Come Funziona (3 step: registrati → timbra → premia)
- Chi Siamo con features grid
- Servizi e prezzi
- Team / Operatori
- Recensioni clienti (con approvazione admin) + riepilogo media stelle e barre distribuzione
- Orari di apertura (evidenzia giorno corrente)
- Contatti
- Sezione Pricing (Base €19 / Pro €29 / Prenoty €49)
- Bottone "Prenota Ora" (solo negozi Prenoty)
- Multi-lingua IT/EN (estensibile)
- SEO meta tags (Open Graph, Twitter Card, favicon)
- Accessibilità (skip-link, aria-labels)

### Area Cliente (integrata nel sito vetrina) — ✅ prototipo completato
- QR Code personale per timbratura rapida al banco
- Carta timbri digitale con animazioni
- Livelli fedeltà Bronze / Silver / Gold
- Storico premi con livello associato
- Form per lasciare recensione (post-premio)
- Google Wallet integration (placeholder)
- Notifiche push (placeholder)

### Pannello Admin — ✅ prototipo completato
- Dashboard Analytics — 4 KPI animate con change %, grafici timbri settimanali, donut stato carte, clienti mensili, distribuzione livelli
- Gestione Clienti — tabella con barra timbri visuale, badge livello, ricerca, timbra/premio con popup conferma
- QR Scanner — viewport con scan line animata, corner markers, scansione istantanea
- Gestione Recensioni — filtri (tutte/in attesa/approvate), approvazione/eliminazione
- Notifiche Push — invio manuale + toggle notifiche automatiche
- Impostazioni complete — 5 sezioni: info attività, colori brand live, carta fedeltà, livelli, sicurezza
- Sezione Prenoty (solo abbonati) — calendario, prenotazioni, statistiche
- **Logica Prenoty/Standalone**: il pannello mostra/nasconde le sezioni Prenoty in base allo stato del negozio

### Personalizzazione Autonoma (stessa codebase per tutti)
Ogni negoziante personalizza in autonomia dal pannello admin — **nessun sito viene creato manualmente**:
- Logo e foto del negozio
- Colori brand (primario, accento) con color picker live
- Nome, tipo e descrizione attività
- Servizi con prezzi
- Operatori / team
- Orari di apertura
- Contatti
- Impostazioni carta fedeltà e livelli
- Per negozi Prenoty: questi dati vengono pre-popolati dal DB Prenoty

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
├── sito-vetrina.html                # ✅ Sito vetrina + area cliente + admin semplificato
├── admin-panel-v2.html              # ✅ Pannello admin completo (stile Shadcn)
├── api/                             # Backend API (da implementare — Fase 1)
│   ├── server.js
│   ├── routes/
│   ├── middleware/
│   │   └── resolveShop.js           # Middleware multi-tenant (slug → shop config)
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
- ✅ Admin Panel v2 — `admin-panel-v2.html`
- ✅ Sito Vetrina + Area Cliente — `sito-vetrina.html`
- 🔄 Super Admin Panel — da creare

---

## 📄 Licenza

Proprietario — © 2026 Benedetto / Prenoty
