# 📋 Carta Fedeltà 2.0 — Piano di Sviluppo

> Documento operativo per lo sviluppo completo della piattaforma.
> Da usare come riferimento in IntelliJ con Claude Code.

---

## Fase 0 — Setup Progetto e Architettura

### 0.1 Definire architettura backend
- [ ] Decidere framework backend (Node.js/Express, Dart Shelf, Python/FastAPI)
- [ ] Decidere come Prenoty espone i dati (API esistente? accesso diretto DB? nuovo microservizio?)
- [ ] Decidere strategia URL multi-tenant:
  - Opzione A: sottodominio → `nomeattivita.cartafedelta.it`
  - Opzione B: path → `cartafedelta.it/nomeattivita`
  - Opzione C: dominio custom del negoziante
- [ ] Decidere deploy strategy su DigitalOcean (App Platform container vs Droplet VPS)

### 0.2 Setup repository e tooling
- [ ] Creare repo Git
- [ ] Struttura cartelle come da README
- [ ] Configurare ESLint, Prettier
- [ ] Setup `.env.example`
- [ ] Configurare hot-reload per sviluppo locale
- [ ] Setup Docker Compose per dev locale (PostgreSQL + API)

### 0.3 Schema Database PostgreSQL
- [ ] Progettare schema completo (vedi dettaglio sotto)
- [ ] Scrivere `schema.sql`
- [ ] Setup sistema migrazioni (es. node-pg-migrate o Knex)
- [ ] Scrivere seed data per sviluppo

#### Schema DB — Tabelle principali

```
shops
├── id (UUID, PK)
├── slug (VARCHAR UNIQUE — usato nell'URL)
├── name (VARCHAR)
├── type (ENUM: barber, hairdresser, restaurant, shop, gym, cafe, beauty, other)
├── emoji (VARCHAR)
├── description (TEXT)
├── about_title (TEXT)
├── about_text (TEXT)
├── primary_color (VARCHAR)
├── accent_color (VARCHAR)
├── font_main (VARCHAR)
├── logo_url (VARCHAR)
├── photo_url (VARCHAR)
├── admin_password_hash (VARCHAR)
├── is_prenoty (BOOLEAN DEFAULT FALSE)
├── prenoty_shop_id (UUID NULLABLE — FK al DB Prenoty)
├── plan (ENUM: free, base, pro, prenoty)
├── plan_status (ENUM: active, trial, expired, cancelled)
├── stripe_customer_id (VARCHAR NULLABLE)
├── stripe_subscription_id (VARCHAR NULLABLE)
├── max_stamps (INTEGER DEFAULT 10)
├── reward_text (VARCHAR)
├── level_bronze_min (INTEGER DEFAULT 0)
├── level_bronze_reward (VARCHAR)
├── level_silver_min (INTEGER DEFAULT 3)
├── level_silver_reward (VARCHAR)
├── level_gold_min (INTEGER DEFAULT 7)
├── level_gold_reward (VARCHAR)
├── fcm_welcome_enabled (BOOLEAN DEFAULT TRUE)
├── fcm_stamp_enabled (BOOLEAN DEFAULT TRUE)
├── fcm_reward_enabled (BOOLEAN DEFAULT TRUE)
├── fcm_inactive_enabled (BOOLEAN DEFAULT FALSE)
├── fcm_inactive_days (INTEGER DEFAULT 14)
├── created_at (TIMESTAMP)
└── updated_at (TIMESTAMP)

shop_services
├── id (UUID, PK)
├── shop_id (UUID, FK → shops)
├── icon (VARCHAR)
├── name (VARCHAR)
├── description (TEXT)
├── price (VARCHAR)
├── sort_order (INTEGER)
└── created_at (TIMESTAMP)

shop_operators
├── id (UUID, PK)
├── shop_id (UUID, FK → shops)
├── name (VARCHAR)
├── role (VARCHAR)
├── emoji (VARCHAR)
├── photo_url (VARCHAR NULLABLE)
├── sort_order (INTEGER)
└── created_at (TIMESTAMP)

shop_schedule
├── id (UUID, PK)
├── shop_id (UUID, FK → shops)
├── day_of_week (INTEGER 0-6)
├── open_time (TIME NULLABLE)
├── close_time (TIME NULLABLE)
├── is_closed (BOOLEAN DEFAULT FALSE)
└── created_at (TIMESTAMP)

shop_contacts
├── id (UUID, PK)
├── shop_id (UUID, FK → shops)
├── type (ENUM: address, phone, email, website, instagram, facebook, whatsapp)
├── icon (VARCHAR)
├── label (VARCHAR)
├── value (VARCHAR)
├── sort_order (INTEGER)
└── created_at (TIMESTAMP)

clients
├── id (UUID, PK)
├── shop_id (UUID, FK → shops)
├── name (VARCHAR)
├── phone (VARCHAR)
├── stamps (INTEGER DEFAULT 0)
├── total_stamps (INTEGER DEFAULT 0)
├── rewards (INTEGER DEFAULT 0)
├── fcm_token (VARCHAR NULLABLE)
├── prenoty_client_id (UUID NULLABLE — FK al DB Prenoty)
├── last_visit (TIMESTAMP)
├── created_at (TIMESTAMP)
└── updated_at (TIMESTAMP)
-- UNIQUE(shop_id, phone)

stamp_history
├── id (UUID, PK)
├── client_id (UUID, FK → clients)
├── shop_id (UUID, FK → shops)
├── stamped_by (VARCHAR — admin name o "qr")
├── created_at (TIMESTAMP)

reward_history
├── id (UUID, PK)
├── client_id (UUID, FK → clients)
├── shop_id (UUID, FK → shops)
├── reward_text (VARCHAR)
├── level (ENUM: bronze, silver, gold)
├── created_at (TIMESTAMP)

reviews
├── id (UUID, PK)
├── shop_id (UUID, FK → shops)
├── client_id (UUID, FK → clients)
├── author_name (VARCHAR)
├── stars (INTEGER 1-5)
├── text (TEXT)
├── is_approved (BOOLEAN DEFAULT FALSE)
├── created_at (TIMESTAMP)

notifications_log
├── id (UUID, PK)
├── shop_id (UUID, FK → shops)
├── title (VARCHAR)
├── body (TEXT)
├── type (ENUM: manual, welcome, stamp, reward, inactive)
├── recipients_count (INTEGER)
├── sent_at (TIMESTAMP)

super_admins
├── id (UUID, PK)
├── email (VARCHAR UNIQUE)
├── password_hash (VARCHAR)
├── name (VARCHAR)
├── created_at (TIMESTAMP)
```

---

## Fase 1 — Infrastruttura Base (API + DB + Multi-tenant)

> PRIORITÀ MASSIMA — tutto il resto dipende da questo.

### 1.1 Backend API — Setup
- [ ] Inizializzare progetto Node.js (o framework scelto)
- [ ] Configurare connessione PostgreSQL (connection pool)
- [ ] Configurare connessione DB Prenoty (read-only, per negozi integrati)
- [ ] Setup CORS, rate limiting, helmet, compression
- [ ] Setup logging (winston/pino)
- [ ] Setup error handling middleware

### 1.2 API — Autenticazione
- [ ] `POST /api/auth/register` — registra nuovo cliente (per shop specifico)
- [ ] `POST /api/auth/login` — login cliente con telefono (ritorna JWT)
- [ ] `POST /api/auth/admin-login` — login admin con password shop
- [ ] `POST /api/auth/superadmin-login` — login super admin
- [ ] Middleware JWT per proteggere le route
- [ ] Middleware `resolveShop` — da slug URL identifica lo shop e carica config

### 1.3 API — Shop Config (multi-tenant)
- [ ] `GET /api/shop/:slug` — ritorna config completa del negozio (dati pubblici)
- [ ] `GET /api/shop/:slug/services` — lista servizi
- [ ] `GET /api/shop/:slug/operators` — lista operatori
- [ ] `GET /api/shop/:slug/schedule` — orari
- [ ] `GET /api/shop/:slug/contacts` — contatti
- [ ] `GET /api/shop/:slug/reviews` — recensioni approvate
- [ ] `GET /api/shop/:slug/stats` — statistiche pubbliche (n. clienti, timbri, premi)

### 1.4 API — Admin CRUD
- [ ] `PUT /api/admin/shop` — aggiorna info negozio, colori, impostazioni
- [ ] `POST/PUT/DELETE /api/admin/services` — CRUD servizi
- [ ] `POST/PUT/DELETE /api/admin/operators` — CRUD operatori
- [ ] `PUT /api/admin/schedule` — aggiorna orari
- [ ] `POST/PUT/DELETE /api/admin/contacts` — CRUD contatti
- [ ] `PUT /api/admin/card-settings` — aggiorna max timbri, premi, livelli
- [ ] `PUT /api/admin/password` — cambio password admin
- [ ] Upload immagini (logo, foto) → salvataggio in DigitalOcean Spaces o locale

### 1.5 API — Clienti & Timbri
- [ ] `GET /api/admin/clients` — lista clienti con ricerca/filtro/paginazione
- [ ] `GET /api/admin/clients/:id` — dettaglio cliente
- [ ] `POST /api/admin/clients/:id/stamp` — aggiungi timbro (con popup conferma lato FE)
- [ ] `POST /api/admin/clients/:id/reward` — applica premio (reset carta, incrementa rewards)
- [ ] `GET /api/client/me` — dati cliente corrente (carta, livello, storico)
- [ ] `GET /api/client/me/rewards` — storico premi

### 1.6 API — Integrazione Prenoty
- [ ] Middleware per detect `is_prenoty` sul negozio
- [ ] Mappatura tabelle Prenoty → tabelle carta fedeltà:
  - Prenoty `customers` → `clients` (via `prenoty_client_id`)
  - Prenoty `businesses` → `shops` (via `prenoty_shop_id`)
  - Prenoty `services` → `shop_services`
  - Prenoty `staff` → `shop_operators`
  - Prenoty `schedules` → `shop_schedule`
- [ ] Sync bidirezionale o read-through:
  - Se `is_prenoty=true`, leggere dati dal DB Prenoty e mergerli
  - Dati specifici carta fedeltà (timbri, premi, livelli) sempre nel DB locale
- [ ] Endpoint per attivare integrazione Prenoty su shop standalone esistente
- [ ] Logica di unificazione clienti (match per telefono) quando shop diventa Prenoty

### 1.7 Frontend — Collegare al backend
- [ ] Refactor `index.html` per fare fetch API invece di usare dati JS in-memory
- [ ] Il sito legge lo slug dalla URL e chiama `GET /api/shop/:slug` al caricamento
- [ ] Il CONFIG viene popolato dalla risposta API
- [ ] Login/registrazione chiamano le API auth
- [ ] Tutte le azioni admin chiamano le API admin
- [ ] Gestione sessione con JWT in localStorage (o cookie HttpOnly)

---

## Fase 2 — QR Code Timbratura

### 2.1 Generazione QR
- [ ] Integrare libreria QR code reale (qrcode.js o qr-code-styling)
- [ ] Ogni cliente ha un QR unico contenente: `{ shopSlug, clientId, token }`
- [ ] Il QR è visualizzato nell'area cliente
- [ ] Il QR è scaricabile come immagine

### 2.2 Scanner QR (Admin)
- [ ] Integrare libreria scanner (html5-qrcode o jsQR)
- [ ] Accesso fotocamera del dispositivo admin
- [ ] Decodifica QR → identificazione cliente
- [ ] Mostra info cliente + bottone "Timbra Adesso"
- [ ] `POST /api/admin/stamp-by-qr` — timbra via QR (con verifica token)
- [ ] Feedback sonoro/vibrazione al successo
- [ ] Gestione errori (QR invalido, cliente non trovato, QR di altro negozio)

---

## Fase 3 — Sistema Recensioni

### 3.1 Scrittura recensioni (cliente)
- [ ] Form recensione visibile solo a clienti con almeno 1 premio
- [ ] Selezione stelle (1-5) + testo
- [ ] `POST /api/client/reviews` — invia recensione
- [ ] Notifica admin di nuova recensione (opzionale)
- [ ] Rate limiting (1 recensione ogni 30 giorni per cliente)

### 3.2 Gestione recensioni (admin)
- [ ] `GET /api/admin/reviews` — tutte le recensioni (approvate + in attesa)
- [ ] `PUT /api/admin/reviews/:id/approve` — approva
- [ ] `DELETE /api/admin/reviews/:id` — elimina
- [ ] Le recensioni approvate appaiono nel sito vetrina

### 3.3 Widget recensioni (pubblico)
- [ ] Sezione "Recensioni" nel sito vetrina
- [ ] Mostra solo approvate, ordinate per data
- [ ] Media stelle visibile nell'hero o vicino ai servizi

---

## Fase 4 — Multi-lingua

### 4.1 Sistema i18n
- [ ] Estendere l'oggetto `I18N` con tutte le stringhe mancanti
- [ ] Supportare lingue aggiuntive (DE, FR, ES, AR...)
- [ ] Toggle lingua nella navbar (attualmente IT/EN)
- [ ] Salvare preferenza lingua in localStorage
- [ ] Le stringhe personalizzate dall'admin (nome servizi, descrizioni) non vengono tradotte — sono inserite come sono
- [ ] Possibilità per l'admin di inserire traduzioni manuali per i propri contenuti (fase futura)

### 4.2 Contenuti traducibili
- [ ] Stringhe UI (navigazione, bottoni, label, messaggi)
- [ ] Stringhe di sistema (notifiche automatiche, email)
- [ ] NON tradotti: nomi servizi, descrizioni, operatori (contenuti del negoziante)

---

## Fase 5 — Livelli Fedeltà (Bronze/Silver/Gold)

### 5.1 Logica livelli
- [ ] Il livello è calcolato sul numero di premi totali riscattati
- [ ] Soglie configurabili dall'admin (default: Bronze 0+, Silver 3+, Gold 7+)
- [ ] Ogni livello ha un premio diverso configurabile (es. Bronze 5%, Silver 10%, Gold 15%)
- [ ] Badge livello visibile nell'area cliente e nella tabella admin
- [ ] Quando un cliente sale di livello → notifica push (se abilitata)

### 5.2 Dashboard livelli
- [ ] Donut chart distribuzione livelli nella dashboard admin
- [ ] Filtro clienti per livello nella tabella

### 5.3 Premi graduati
- [ ] Quando l'admin applica un premio, il premio è quello del livello corrente del cliente
- [ ] Storico premi mostra il livello al momento del riscatto

---

## Fase 6 — Prenotazione Rapida dal Sito (Solo Prenoty)

### 6.1 Bottone "Prenota Ora"
- [ ] Visibile solo se `is_prenoty = true`
- [ ] Sezione dedicata nel sito vetrina (tra orari e contatti)
- [ ] CTA prominente nell'hero

### 6.2 Flusso prenotazione
- [ ] Opzione A: redirect all'app Prenoty / pagina web Prenoty
- [ ] Opzione B: widget prenotazione embedded nel sito (iframe o componente)
- [ ] Opzione C: form prenotazione integrato che scrive direttamente nel DB Prenoty
- [ ] Da decidere in base a come Prenoty gestisce le prenotazioni

### 6.3 Pannello admin Prenoty
- [ ] Sezione "Prenoty" nel pannello admin
- [ ] Vista calendario prenotazioni (read dal DB Prenoty)
- [ ] Lista prenotazioni del giorno
- [ ] Statistiche prenotazioni

---

## Fase 7 — Super Admin Panel

### 7.1 Autenticazione super admin
- [ ] Login separato con email + password
- [ ] JWT con ruolo `superadmin`
- [ ] Protezione route API con middleware ruolo

### 7.2 Dashboard piattaforma
- [ ] `GET /api/superadmin/stats` — metriche globali:
  - Totale negozi
  - Negozi Prenoty vs standalone
  - MRR (Monthly Recurring Revenue)
  - Utenti totali (somma clienti tutti i negozi)
  - Nuovi negozi questo mese
  - Churn rate

### 7.3 Gestione negozi
- [ ] `GET /api/superadmin/shops` — lista tutti i negozi con filtri
- [ ] `GET /api/superadmin/shops/:id` — dettaglio negozio
- [ ] `PUT /api/superadmin/shops/:id` — modifica piano, stato, note
- [ ] `DELETE /api/superadmin/shops/:id` — disattiva negozio
- [ ] Possibilità di impersonare un admin per supporto

### 7.4 Onboarding nuovi negozi
- [ ] `POST /api/superadmin/shops` — crea nuovo negozio manualmente
- [ ] Oppure self-service: il negoziante si registra da solo (pagina pricing pubblica)
- [ ] Wizard di setup (nome, tipo, servizi base)

---

## Fase 8 — Stripe / Pagamenti

### 8.1 Setup Stripe
- [ ] Creare prodotti e piani in Stripe Dashboard:
  - Base: €19/mese
  - Pro: €29/mese
  - Prenoty: €49/mese (o gratis se gestito lato Prenoty)
- [ ] Configurare webhook Stripe

### 8.2 Checkout
- [ ] `POST /api/stripe/create-checkout` — crea sessione Stripe Checkout
- [ ] Pagina pricing pubblica con bottoni "Scegli Piano"
- [ ] Redirect a Stripe Checkout → ritorno a success/cancel page

### 8.3 Webhook e gestione abbonamenti
- [ ] `POST /api/stripe/webhook` — gestisce eventi Stripe:
  - `checkout.session.completed` → attiva shop
  - `invoice.paid` → rinnovo ok
  - `invoice.payment_failed` → notifica admin
  - `customer.subscription.deleted` → disattiva shop
- [ ] Aggiornare `plan_status` in DB in base agli eventi

### 8.4 Customer portal
- [ ] `POST /api/stripe/portal` — link al Stripe Customer Portal
- [ ] Il negoziante può gestire fatture, carta, cancellazione da Stripe

---

## Fase 9 — Notifiche Push (Firebase FCM)

### 9.1 Setup Firebase
- [ ] Creare progetto Firebase (o usare quello di Prenoty)
- [ ] Ottenere credenziali server (service account key)
- [ ] Configurare Firebase Admin SDK nel backend

### 9.2 Registrazione token
- [ ] Il client (browser) richiede permesso notifiche
- [ ] Ottiene FCM token tramite Firebase SDK client
- [ ] `POST /api/client/fcm-token` — salva token nel DB

### 9.3 Invio notifiche
- [ ] Servizio backend per invio notifiche:
  - A singolo client (per token)
  - A tutti i clienti di un negozio (batch)
  - A topic (es. `shop_<shopId>`)
- [ ] Notifiche automatiche (trigger su eventi DB):
  - Registrazione → benvenuto
  - Nuovo timbro → conferma
  - Carta completa → premio disponibile
  - Inattività → promemoria
- [ ] `POST /api/admin/notifications/send` — invio manuale
- [ ] Log notifiche inviate in `notifications_log`

### 9.4 Service Worker
- [ ] Creare `firebase-messaging-sw.js`
- [ ] Gestione notifiche in background
- [ ] Click su notifica → apre il sito del negozio

---

## Fase 10 — Google Wallet Integration

### 10.1 Setup Google Wallet API
- [ ] Creare account Google Wallet API su Google Cloud Console
- [ ] Configurare Loyalty Class (template carta fedeltà)
- [ ] Generare chiave di servizio

### 10.2 Generazione pass
- [ ] `POST /api/client/wallet-pass` — genera JWT per Google Wallet
- [ ] Il pass contiene: nome negozio, logo, nome cliente, timbri attuali, livello
- [ ] Link "Add to Google Wallet" nell'area cliente

### 10.3 Aggiornamento pass
- [ ] Quando il cliente riceve un timbro o premio, il pass si aggiorna automaticamente
- [ ] Webhook Google Wallet per sync

---

## Fase 11 — Deploy e Go-Live

### 11.1 DigitalOcean Setup
- [ ] Creare Managed PostgreSQL Database
- [ ] Configurare connessione sicura (SSL)
- [ ] Decidere hosting:
  - App Platform (container Docker)
  - Droplet con PM2 + Nginx
- [ ] Configurare dominio e SSL (Let's Encrypt)

### 11.2 Multi-tenant URL
- [ ] Configurare wildcard DNS se sottodomini
- [ ] Oppure routing path-based con Nginx/API
- [ ] Supporto dominio custom (opzionale, CNAME)

### 11.3 CI/CD
- [ ] GitHub Actions per build + test + deploy automatico
- [ ] Migrazioni DB automatiche in pipeline

### 11.4 Monitoring
- [ ] Setup health check endpoint
- [ ] Logging centralizzato
- [ ] Alerting errori (Sentry o simile)
- [ ] Monitoraggio performance DB

---

## Fase 12 — Polish e Ottimizzazioni

### 12.1 Performance
- [ ] Lazy loading immagini
- [ ] Compressione assets (gzip/brotli)
- [ ] Caching API con Redis (opzionale)
- [ ] CDN per assets statici

### 12.2 SEO
- [ ] Meta tag dinamici per ogni negozio
- [ ] Open Graph tags per condivisione social
- [ ] Sitemap.xml dinamico
- [ ] robots.txt

### 12.3 Accessibilità
- [ ] Aria labels su tutti gli elementi interattivi
- [ ] Navigazione da tastiera
- [ ] Contrasto colori adeguato (WCAG AA)
- [ ] Screen reader friendly

### 12.4 Sicurezza
- [ ] Rate limiting API
- [ ] Input sanitization
- [ ] CSRF protection
- [ ] Password hashing (bcrypt)
- [ ] SQL injection prevention (parametrized queries)
- [ ] HTTPS everywhere

---

## Note per Claude Code

Quando lavori su questo progetto con Claude Code in IntelliJ:

1. **Parti dalla Fase 0** — definisci architettura e schema DB prima di tutto
2. **Una fase alla volta** — non saltare fasi, ogni fase dipende dalla precedente
3. **Il frontend (`index.html`) è il prototipo** — serve come riferimento visivo, va refactorato per chiamare le API reali
4. **Il DB PostgreSQL è condiviso con Prenoty** — attenzione a non modificare tabelle Prenoty, usare solo lettura
5. **Testa sempre in locale** prima di deployare
6. **Ogni endpoint API va con test** — almeno happy path + errore principale

### Comandi utili
```bash
# Avvia server dev
npm run dev

# Esegui migrazioni
npm run migrate

# Seed dati demo
npm run seed

# Esegui test
npm test

# Build per produzione
npm run build
```

---

> Ultimo aggiornamento: 21 Marzo 2026
