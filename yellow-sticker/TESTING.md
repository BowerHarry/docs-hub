# Testing and operations

## Monitor dashboard (`/monitor`)

The internal dashboard is intended for operators, not end users. Typical uses:

- Send lifecycle email samples via **`send-test-email`**.
- **Preview cancel** — read-only simulation of cancel + Stripe + email side effects (`admin-preview-cancel`).
- **Add production** — poster upload and metadata (`admin-create-production`).
- **Test fixture** — drives a hidden `test-fixture` production through reset → simulate availability → mark tickets found → clear alert state → delete, so you can exercise signup → alert → cancel without touching real shows (`admin-test-fixture`).

## Extension

Use **Run once now** on the extension options page after changing Supabase URL, anon key, or `SCRAPER_SHARED_SECRET`.

## Local end-to-end checks

`scripts/dev/local-stack.sh start` gives a local backend with demo data and Stripe test mode (see [`DEVELOPMENT.md`](./DEVELOPMENT.md)). Worth running by hand after touching payments or alerts:

- subscribe, pay with `4242 4242 4242 4242`, and confirm the row becomes `paid` and every webhook delivery in `supabase/.temp/local/stripe.log` is a 200;
- `stripe events resend <evt_id>` and confirm the function logs "Duplicate webhook event";
- cancel with refund from the manage page, then repeat the request and confirm a 409;
- POST `available`, `error`, `available` to `report-scrape` and confirm only the first is a `transition`.

## Automated tests

There are none yet. Add them here as they are introduced (Playwright, integration against the local stack, etc.).
