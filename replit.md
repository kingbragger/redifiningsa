# Redefining Travel SA

A regenerative tourism marketplace connecting impact-driven travelers with rural South African communities through immersive cultural experiences and an artisan marketplace.

## Architecture

- **Frontend**: React 18 + Vite + TypeScript, using Tailwind CSS + shadcn/ui components
- **Backend**: Express.js server (`server/index.ts`) running on port 3001 (dev) / 5000 (prod)
- **Auth**: Supabase — handles user authentication only
- **Database**: Replit PostgreSQL — stores all bookings, hosts, experiences, orders, rewards, vouchers
- **AI Chatbot**: `/api/ai-support` calls OpenAI `gpt-4o-mini` with tourism system prompt (Thandi)
- **Payments**: PayPal SDK (live by default via `PAYPAL_ENV=live`); also EFT with proof-of-payment upload

## Database Tables (Replit PostgreSQL)

All auto-created on server start via `initDb()`:

| Table | Purpose |
|---|---|
| `bookings` | Experience bookings with EFT proof upload, payment/status tracking, feedback_message |
| `hosts` | Legacy host registrations |
| `host_applications` | Full applications with profile image, ID doc, lat/lng, feedback_message |
| `experiences` | Cultural experiences created in admin |
| `orders` | Artisan marketplace orders with EFT/PayPal, order_status, feedback_message |
| `contact_messages` | Website contact form submissions |
| `admin_users` | Admin accounts with bcrypt-hashed passwords |
| `user_rewards` | Customer loyalty points and tier (Bronze/Silver/Gold/Platinum) |
| `voucher_purchases` | Voucher requests via EFT; admin assigns unique code on approval |

## Key Routes

### Customer-facing
- `/` — Homepage
- `/book` — Experience booking
- `/marketplace` — Artisan marketplace with PayPal + EFT checkout
- `/track-order` — Order/booking tracker by email + reference
- `/my-dashboard` — Customer self-service: orders, bookings, host app status, rewards, vouchers
- `/host-register` — Host application with profile image, ID doc, map
- `/auth` — Supabase login/signup

### Admin
- `/admin/login` — JWT login
- `/admin` — Admin dashboard (Bookings, Orders, Experiences, Hosts, Messages, Vouchers, Rewards tabs)

## Admin Dashboard

Access: `admin@redefiningtravel.co.za` / `RTravel@Admin2024!`
Also: `info@redefiningtravelsa.co.za` (same password)

Features:
- Stats bar: Bookings, Orders, Pending Hosts, Experiences, Revenue, Unread Messages, Pending Vouchers
- Tabs: Bookings, Orders, Experiences, Hosts, Messages, Vouchers, Rewards
- Inline feedback textarea on every order/booking/host-app card (visible to customer in My Dashboard)
- Voucher approval: generates unique `RTSA-XXXXXX` code
- Rewards: award points to customers by email

## My Dashboard (Customer Self-Service)

Route: `/my-dashboard` — no login required, just email lookup.
Tabs:
- **Orders** — marketplace order history with status + admin feedback
- **Bookings** — experience bookings with status + admin feedback
- **Host App** — shows current status + admin feedback
- **Vouchers** — requested vouchers + approved codes
- **Rewards** — loyalty points, tier badge, progress bar
- **Buy Voucher** — EFT voucher request with proof upload

## Payment Methods

- **PayPal** — live PayPal SDK; ZAR→USD at 0.055 rate
- **Bank EFT** — Capitec bank (Lucy Kgware, acc: 1687960850, branch: 470010); use email as reference
- **Voucher codes** — WELCOME10 (10%), SAVE50 (flat R50), COMMUNITY20 (20%), plus admin-generated RTSA-* codes

## Rewards Tiers

| Level | Min Points | Benefits |
|---|---|---|
| Bronze | 0 | Welcome |
| Silver | 500 | 5% discount |
| Gold | 2000 | 10% discount + priority support |
| Platinum | 5000 | 15% discount + exclusive experiences |

## Running the App

```bash
npm run dev
```

Runs Vite (port 5000) + Express (port 3001) concurrently via workflow "Start application".

## Production Build

```bash
vite build && esbuild server/index.ts --platform=node --packages=external --bundle --format=cjs --outfile=dist/index.cjs
node dist/index.cjs
```

## Environment Secrets

| Secret | Purpose |
|---|---|
| `DATABASE_URL` | Replit PostgreSQL connection string |
| `OPENAI_API_KEY` | AI chatbot Thandi (gpt-4o-mini) |
| `PAYPAL_CLIENT_ID` | PayPal client ID |
| `PAYPAL_CLIENT_SECRET` | PayPal client secret |
| `JWT_SECRET` | Signs admin auth cookies |
| `SUPABASE_JWT_SECRET` | Supabase JWT secret |
| `SUPABASE_SERVICE_ROLE_KEY` | Supabase service role key |

## File Uploads

- Stored in `<cwd>/uploads/` directory
- Served as static files at `/uploads/*`
- multer `upload` — proof of payment files (EFT)
- multer `uploadHostApp` — host profile images + ID docs (fields: `profileImage`, `idDocument`)
