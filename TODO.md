# 📋 Carta Fedeltà 2.0 — Piano di Sviluppo

> Documento operativo per lo sviluppo completo della piattaforma.
> Da usare come riferimento in IntelliJ con Claude Code.

---

## ⚡ Stato Attuale

### Completato
- ✅ Admin Panel v2 — prototipo frontend completo (`prototypes/admin-panel-v2.html`)
  - Dashboard con 4 KPI, 4 grafici (barre + donut)
  - Gestione clienti con tabella, timbri, livelli, azioni
  - QR Scanner con viewport animato
  - Recensioni con filtri e approvazione
  - Notifiche push (invio manuale + toggle automatiche)
  - Impostazioni complete (5 sezioni)
  - Sezione Prenoty (nascondibile)
  - Design: stile TailAdmin/Shadcn, dark mode, Geist font, animazioni stagger

### Da completare (frontend)
- 🔄 Sito Vetrina — da rifare con lo stesso stile Shadcn dell'admin
- 🔄 Area Cliente — QR code, carta timbri, livelli, recensioni (stile Shadcn)
- 🔄 Super Admin Panel — metriche piattaforma, tabella negozi, pricing (stile Shadcn)
- 🔄 Unificare tutto in un unico file o modularizzare in file separati

### Decisioni di design prese
- Stile: TailAdmin/Shadcn (clean, sharp, enterprise) + tocchi premium (blur, ombre morbide)
- Animazioni: tutte (stagger, hover, grafici animati, confetti, skeleton, transizioni)
- Tema: dark mode, gerarchia superfici, Geist + Geist Mono
- Colori: indigo primario, palette accenti per KPI

### Domande architetturali ancora aperte
- Backend framework (Node/Express? Dart server? Python/FastAPI?)
- Come Prenoty espone i dati (API esistente? accesso diretto DB?)
- Strategia URL multi-tenant (sottodominio? path? dominio custom?)
- Deploy DigitalOcean (App Platform? Droplet?)

---

## Fase 0 — Setup Progetto e Architettura

### 0.1 Definire architettura backend
- [ ] Decidere framework backend
- [ ] Decidere come Prenoty espone i dati
- [ ] Decidere strategia URL multi-tenant
- [ ] Decidere deploy strategy su DigitalOcean

### 0.2 Setup repository e tooling
- [ ] Struttura cartelle definitiva
- [ ] Configurare ESLint, Prettier
- [ ] Setup `.env.example`
- [ ] Setup Docker Compose per dev locale (PostgreSQL + API)

### 0.3 Schema Database PostgreSQL
- [ ] Scrivere `schema.sql` (vedi dettaglio sotto)
- [ ] Setup migrazioni
- [ ] Seed data per sviluppo

#### Schema DB — Tabelle principali

```sql
-- shops: config di ogni negozio
shops (
  id UUID PK,
  slug VARCHAR UNIQUE,          -- URL identifier
  name VARCHAR,
  type ENUM(barber,hairdresser,restaurant,shop,gym,cafe,beauty,other),
  emoji VARCHAR,
  description TEXT,
  about_title TEXT,
  about_text TEXT,
  primary_color VARCHAR DEFAULT '#6366F1',
  accent_color VARCHAR DEFAULT '#8B5CF6',
  font_main VARCHAR DEFAULT 'Geist',
  logo_url VARCHAR,
  photo_url VARCHAR,
  admin_password_hash VARCHAR,
  is_prenoty BOOLEAN DEFAULT FALSE,
  prenoty_shop_id UUID NULLABLE,
  plan ENUM(free,base,pro,prenoty),
  plan_status ENUM(active,trial,expired,cancelled),
  stripe_customer_id VARCHAR,
  stripe_subscription_id VARCHAR,
  max_stamps INTEGER DEFAULT 10,
  reward_text VARCHAR DEFAULT 'Sconto 10%',
  level_bronze_min INTEGER DEFAULT 0,
  level_bronze_reward VARCHAR DEFAULT 'Sconto 5%',
  level_silver_min INTEGER DEFAULT 3,
  level_silver_reward VARCHAR DEFAULT 'Sconto 10%',
  level_gold_min INTEGER DEFAULT 7,
  level_gold_reward VARCHAR DEFAULT 'Sconto 15%',
  fcm_welcome_enabled BOOLEAN DEFAULT TRUE,
  fcm_stamp_enabled BOOLEAN DEFAULT TRUE,
  fcm_reward_enabled BOOLEAN DEFAULT TRUE,
  fcm_inactive_enabled BOOLEAN DEFAULT FALSE,
  fcm_inactive_days INTEGER DEFAULT 14,
  created_at TIMESTAMP,
  updated_at TIMESTAMP
)

-- shop_services, shop_operators, shop_schedule, shop_contacts
-- (vedi README per struttura completa)

-- clients: clienti di ogni negozio
clients (
  id UUID PK,
  shop_id UUID FK → shops,
  name VARCHAR,
  phone VARCHAR,
  stamps INTEGER DEFAULT 0,
  total_stamps INTEGER DEFAULT 0,
  rewards INTEGER DEFAULT 0,
  fcm_token VARCHAR,
  prenoty_client_id UUID NULLABLE,
  last_visit TIMESTAMP,
  created_at TIMESTAMP,
  UNIQUE(shop_id, phone)
)

-- stamp_history, reward_history, reviews, notifications_log, super_admins
-- (vedi TODO dettagliato originale per struttura completa)
```

---

## Fase 1 — Infrastruttura Base (API + DB + Multi-tenant)

### 1.1 Backend API — Setup
- [ ] Inizializzare progetto
- [ ] Connessione PostgreSQL (pool)
- [ ] Connessione DB Prenoty (read-only)
- [ ] CORS, rate limiting, helmet, compression
- [ ] Logging, error handling

### 1.2 API — Autenticazione
- [ ] `POST /api/auth/register` — registra cliente
- [ ] `POST /api/auth/login` — login cliente (telefono → JWT)
- [ ] `POST /api/auth/admin-login` — login admin
- [ ] `POST /api/auth/superadmin-login` — login super admin
- [ ] Middleware JWT + middleware `resolveShop` (da slug)

### 1.3 API — Shop Config (multi-tenant)
- [ ] `GET /api/shop/:slug` — config completa negozio
- [ ] `GET /api/shop/:slug/services`
- [ ] `GET /api/shop/:slug/operators`
- [ ] `GET /api/shop/:slug/schedule`
- [ ] `GET /api/shop/:slug/contacts`
- [ ] `GET /api/shop/:slug/reviews` — solo approvate
- [ ] `GET /api/shop/:slug/stats` — statistiche pubbliche

### 1.4 API — Admin CRUD
- [ ] `PUT /api/admin/shop` — aggiorna tutto
- [ ] CRUD servizi, operatori, contatti
- [ ] `PUT /api/admin/schedule`
- [ ] `PUT /api/admin/card-settings`
- [ ] `PUT /api/admin/password`
- [ ] Upload immagini

### 1.5 API — Clienti & Timbri
- [ ] `GET /api/admin/clients` — lista con ricerca/paginazione
- [ ] `POST /api/admin/clients/:id/stamp`
- [ ] `POST /api/admin/clients/:id/reward`
- [ ] `GET /api/client/me` — dati cliente corrente
- [ ] `GET /api/client/me/rewards`

### 1.6 API — Integrazione Prenoty
- [ ] Middleware detect `is_prenoty`
- [ ] Mappatura tabelle Prenoty → tabelle locali
- [ ] Sync/read-through dati Prenoty
- [ ] Endpoint attivazione integrazione
- [ ] Unificazione clienti (match per telefono)

### 1.7 Frontend — Collegare al backend
- [ ] Refactor frontend per fetch API
- [ ] Il sito legge slug da URL → `GET /api/shop/:slug`
- [ ] Login/registrazione → API auth
- [ ] Azioni admin → API admin
- [ ] Sessione con JWT

---

## Fase 2 — QR Code Timbratura

- [ ] Libreria QR reale (qrcode.js / qr-code-styling)
- [ ] QR unico per cliente: `{ shopSlug, clientId, token }`
- [ ] QR visibile + scaricabile nell'area cliente
- [ ] Scanner admin (html5-qrcode / jsQR)
- [ ] `POST /api/admin/stamp-by-qr`
- [ ] Feedback sonoro/vibrazione
- [ ] Gestione errori (QR invalido, altro negozio)

---

## Fase 3 — Sistema Recensioni

- [ ] Form visibile solo a clienti con ≥1 premio
- [ ] `POST /api/client/reviews`
- [ ] Rate limiting (1 ogni 30 giorni)
- [ ] `GET /api/admin/reviews` — tutte
- [ ] `PUT /api/admin/reviews/:id/approve`
- [ ] `DELETE /api/admin/reviews/:id`
- [ ] Media stelle nel sito vetrina

---

## Fase 4 — Multi-lingua

- [ ] Oggetto i18n con tutte le stringhe (IT, EN + estensibile)
- [ ] Toggle lingua nella navbar
- [ ] Salva preferenza in localStorage
- [ ] Contenuti admin non tradotti (inseriti come sono)

---

## Fase 5 — Livelli Fedeltà (Bronze/Silver/Gold)

- [ ] Calcolo livello su numero premi totali
- [ ] Soglie configurabili dall'admin
- [ ] Premi diversi per livello
- [ ] Notifica push al cambio livello
- [ ] Donut distribuzione livelli in dashboard
- [ ] Filtro clienti per livello

---

## Fase 6 — Prenotazione Rapida (Solo Prenoty)

- [ ] Bottone "Prenota Ora" visibile solo se `is_prenoty=true`
- [ ] Decidere flusso: redirect / widget / form integrato
- [ ] Vista calendario prenotazioni nell'admin
- [ ] Lista prenotazioni del giorno

---

## Fase 7 — Super Admin Panel

- [ ] Login separato email + password
- [ ] `GET /api/superadmin/stats` — metriche globali
- [ ] `GET /api/superadmin/shops` — lista negozi
- [ ] `PUT /api/superadmin/shops/:id` — modifica piano/stato
- [ ] Onboarding nuovi negozi (manuale o self-service)
- [ ] Possibilità di impersonare admin per supporto

---

## Fase 8 — Stripe / Pagamenti

- [ ] Prodotti Stripe: Base €19, Pro €29, Prenoty €49
- [ ] `POST /api/stripe/create-checkout`
- [ ] Pagina pricing pubblica
- [ ] Webhook: checkout.completed, invoice.paid/failed, subscription.deleted
- [ ] Customer portal per gestione fatture

---

## Fase 9 — Firebase FCM

- [ ] Setup Firebase Admin SDK
- [ ] Registrazione FCM token dal browser
- [ ] Invio notifiche: singolo client, batch, topic
- [ ] Trigger automatici: registrazione, timbro, carta completa, inattività
- [ ] Service worker `firebase-messaging-sw.js`
- [ ] Log notifiche in `notifications_log`

---

## Fase 10 — Google Wallet

- [ ] Setup Google Wallet API
- [ ] Loyalty Class template
- [ ] `POST /api/client/wallet-pass` — genera JWT
- [ ] Link "Add to Google Wallet" nell'area cliente
- [ ] Aggiornamento automatico pass al timbro/premio

---

## Fase 11 — Deploy

- [ ] Managed PostgreSQL su DigitalOcean
- [ ] Hosting (App Platform / Droplet + Nginx)
- [ ] Dominio + SSL
- [ ] Multi-tenant URL (wildcard DNS o routing)
- [ ] GitHub Actions CI/CD
- [ ] Health check + monitoring

---

## Fase 12 — Polish

- [ ] Lazy loading, compressione, CDN
- [ ] Meta tag SEO dinamici per negozio
- [ ] Open Graph, sitemap
- [ ] Aria labels, keyboard nav, WCAG AA
- [ ] Rate limiting, sanitization, CSRF, bcrypt

---

## 🛠 Note per Claude Code (IntelliJ)

1. **Parti dalla Fase 0** — definisci architettura e schema DB
2. **Una fase alla volta** — ogni fase dipende dalla precedente
3. **I file in `prototypes/` sono riferimento visivo** — da usare per ricostruire il frontend collegato alle API
4. **Il DB PostgreSQL è condiviso con Prenoty** — solo lettura su tabelle Prenoty
5. **Testa in locale** prima di deployare
6. **Lo stile è Shadcn/TailAdmin** — mantieni coerenza con `admin-panel-v2.html`

### Prossimi passi immediati
1. Rispondere alle domande architetturali aperte (backend, URL, deploy)
2. Rifare sito vetrina + area cliente + super admin con stesso stile dell'admin
3. Scrivere schema.sql completo
4. Iniziare backend API

### Comandi utili
```bash
npm run dev        # Avvia server dev
npm run migrate    # Esegui migrazioni DB
npm run seed       # Seed dati demo
npm test           # Test
npm run build      # Build produzione
```

---

> Ultimo aggiornamento: 22 Marzo 2026
