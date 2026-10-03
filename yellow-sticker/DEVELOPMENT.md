# Development and deployment

How to run Supabase locally, deploy Edge Functions and secrets, and ship the web app — aimed at whoever operates this repo (typically you).

## Prerequisites

- Node 18+ for `web/`
- [Supabase CLI](https://supabase.com/docs/guides/cli) for database and edge functions
- Firefox for the `firefox-extension/` scraper (see [`firefox-extension/README.md`](https://github.com/BowerHarry/YellowSticker/blob/main/firefox-extension/README.md))

## Local backend

Two ways to run it. Both are for development only and use Stripe test mode.

**Without Docker** (what the README describes, and what has been run end to end):

```bash
scripts/dev/local-stack.sh start
```

It starts a throwaway Postgres with this repo's migrations, `seed.sql` and
`scripts/dev/seed-demo.sql`, plus PostgREST, the edge functions under Deno and
`stripe listen`. It needs `postgres`, `postgrest`, `deno`, `node` and the
Stripe CLI on your PATH, and skips three legacy migrations that require the
`pg_cron` extension (later migrations remove what they create).

**With Docker**, the standard Supabase CLI route:

```bash
supabase start
supabase db reset   # applies migrations + seed
supabase functions serve --env-file <your test-mode env file>
```

The migration that used to stop this on a fresh database
(`20241114002_setup_cron.sql`) has been fixed, but this route has not been
re-run since; expect to allow several GB of disk for the images.

Monitoring is driven by the Firefox extension posting to `report-scrape`, not
by pg_cron — see [`SECRETS.md`](./SECRETS.md) for the keys each component needs.

## Edge functions

Deploy from the repo root (adjust the function list to match your project):

```bash
supabase functions deploy   # every function under supabase/functions
```

Set secrets (never commit values):

```bash
supabase secrets set SCRAPER_SHARED_SECRET="$(openssl rand -hex 32)"
supabase secrets set BACKEND_API_SECRET_KEY="sb_secret_..."   # Dashboard → API Keys → Secret (custom names cannot start with SUPABASE_)
# Optional: SERVICE_ROLE_KEY for a legacy JWT. The platform may still inject SUPABASE_SERVICE_ROLE_KEY automatically.
# plus RESEND_*, STRIPE_*, etc. — see docs/env.sample
```

## Web SPA

```bash
cd web
npm install
cp env.sample .env.local   # Vite `VITE_*` keys; see comments at top of `web/env.sample`
npm run dev
```

The full backend + operations variable list is in [`env.sample`](./env.sample) (repo root `docs/env.sample` when working from `docs/`).

## Stripe test vs live

Stripe mode follows the **`STRIPE_SECRET_KEY`** prefix (`sk_test_*` vs `sk_live_*`). Test and live objects are not interchangeable; mismatched rows in the database cause confusing cancel/refund behaviour.

Operational checklist (wipe test data before going live, webhook secrets per mode, etc.) lives in [`STRIPE.md`](./STRIPE.md).

## Migrations

- Add new files under `supabase/migrations/` only (do not rewrite applied migrations on shared branches without team agreement), then `supabase db push`.  
- Avoid embedding secrets in SQL; use settings or Vault — [`SECRETS.md`](./SECRETS.md).

## Testing and monitor dashboard

End-to-end checks using the **Test fixture** production and the `/monitor` dashboard are summarized in [`TESTING.md`](./TESTING.md).
