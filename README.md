# Vendordesk

Vendordesk is a small, mobile-first web app to convert chat-based orders into structured records, keep a single source of truth for inventory, and help small vendors avoid lost orders and stock errors. This repo contains the V1 product: Quick Capture (paste/upload), an editable order-confirmation form, order lifecycle management, and an immutable inventory ledger.

## Features (V1)

- Quick Capture: paste chat text or upload a screenshot to pre-fill an order.
- Human-confirmation order form: edit parsed customer and items before saving.
- Product catalog with variants and SKUs.
- Orders list and single-order view (Pending → Confirmed → Fulfilled).
- Inventory ledger: immutable records of every stock change.
- Low-stock notifications and a reconciliation view.
- Mobile-first UI, minimal integrations (no messaging API in V1).

## Tech stack
- Frontend & server functions: Next.js
- Database, auth, storage, realtime: Supabase (Postgres)
- Version control: GitHub
- Hosting / CI: Vercel

## Quick start (prereqs)
- Node.js (recommended LTS)
- npm, yarn, or pnpm
- Supabase account (free tier is fine for V1)
- Vercel account
- GitHub account

## Environment variables (example names)
Store these in a .env.local for local dev and in Vercel project settings for production. Never commit these to Git.

- NEXT_PUBLIC_SUPABASE_URL
- NEXT_PUBLIC_SUPABASE_ANON_KEY
- SUPABASE_SERVICE_ROLE_KEY (server-only key; do not expose to client)
- NEXTAUTH_URL or other session-related envs (if using NextAuth)
- NEXT_PUBLIC_APP_URL (optional, e.g., http://localhost:3000)

Notes:
- The anon key is safe for client usage with Row Level Security (RLS) enabled.
- Use SUPABASE_SERVICE_ROLE_KEY only on server endpoints where you need unrestricted DB access (e.g., migrations, admin tasks).

## Database (minimal schema overview)
Create these core tables for V1. This is a high-level summary — I can produce SQL later if you want.

- vendors
  - id, name, plan, created_at
- products
  - id, vendor_id, sku, title, variant, unit_price, reorder_point, created_at
- inventory (or products.current_stock)
  - product_id, current_stock
- customers
  - id, vendor_id, name, phone, notes
- orders
  - id, vendor_id, customer_id, total_amount, status, created_at, updated_at
- order_items
  - id, order_id, product_id, sku, qty, unit_price
- inventory_ledger
  - id, product_id, change (int), reason (order_id/manual/adj), created_at

Important DB rules:
- Use transactions for order creation + inventory decrement.
- Use Row Level Security (RLS) to scope all rows to vendor_id.
- Keep inventory_ledger immutable to support reconciliation.

Storage:
- A Supabase Storage bucket (e.g., `screenshots`) for uploaded chat images. Store signed URLs and metadata in the order record.

## Local development (high-level)
1. Clone the repo:
   - git clone <your-repo-url>
2. Install dependencies:
   - npm install (or yarn / pnpm)
3. Create a Supabase project and set the env vars listed above locally.
4. Run dev server:
   - npm run dev
5. Open http://localhost:3000 and log in / sign up via Supabase auth.

(If you want a runnable checklist with exact commands, I can produce that next.)

## Deployment
- Connect this GitHub repo to Vercel.
- Add the same environment variables in Vercel project settings.
- Configure Vercel to build the Next.js app; Vercel will create preview URLs for PRs and a production URL for the main branch automatically.
- Ensure Supabase keys used in serverless functions are service-role keys only when necessary, and keep them server-side.

## Authentication & Security
- Use Supabase Auth for vendor login (email/magic link for quick onboarding).
- Enforce Row Level Security in the database so each vendor can only query their own rows.
- Treat customer phone numbers and contact details as personal data (PII): limit retention and provide export/removal options.
- Do not expose service-role keys in client code.

## Data privacy & backups
- Enable Supabase automated backups for your project.
- Document a retention policy for customer data and screenshots.
- Use signed URLs for accessing stored screenshots so they’re not publicly exposed.

## Contributing
- Use feature branches: feature/<brief-description>
- Open a pull request for review before merging to main.
- Keep PRs small and focused.
- Add a short description in PRs about any DB changes.

## Troubleshooting / Common gotchas
- Oversells: make sure server endpoints update inventory in a transaction.
- Missing env vars: the app will fail to connect to Supabase if keys are wrong — re-check Vercel and local .env.
- RLS blocking queries: when RLS is on, use authenticated context or service key for admin tasks.

## Roadmap (beyond V1 — deferred)
- Automated WhatsApp/Instagram connectors
- OCR that fully auto-fills forms (no human review)
- Multi-user roles and audit logs
- Payment/invoicing integrations
- Advanced analytics and forecasting

## License
This project is licensed under the MIT License.

## Contact
- Project owner: bettyid
- For help: open an issue in this repo and tag `help wanted`.
