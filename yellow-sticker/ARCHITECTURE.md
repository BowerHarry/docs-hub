# Architecture

How traffic flows between the web app, Stripe, Supabase edge functions, Resend, Telegram and the browser-extension monitor. Written for maintainers.

## Data flow

```
┌────────────────┐        1. subscribe + pay        ┌─────────────┐
│                │ ───────────────────────────────▶ │             │
│   Web app      │ ◀─ 2. Stripe Checkout redirect ─ │   Stripe    │
│                │                                  │             │
└──────┬─────────┘                                  └──────┬──────┘
       │  reads productions                               │ 3. signed webhook
       ▼                                                  ▼
┌──────────────────────────────────────────────────────────────┐
│                         Supabase                             │
│   Postgres (RLS on)        Edge functions                    │
│   ─ users                  ─ create-checkout-session         │
│   ─ productions            ─ stripe-webhook                  │
│   ─ subscriptions          ─ subscription-management         │
│   ─ notification_logs      ─ request-manage-link             │
│   ─ stripe_events          ─ telegram-webhook                │
│   ─ scrape_heartbeats      ─ report-scrape  ◀─── POST        │
│   ─ scraper_settings       ─ status-dashboard, admin-*       │
└──────────────────────▲───────────────────────────────────────┘
                       │ availability reports
                       │ (shared-secret header)
┌──────────────────────┴───────────────────────────────────────┐
│  firefox-extension/ — the monitor                            │
│  ─ wakes on a timer (every 10 minutes by default)            │
│  ─ checks today's standing-ticket availability per show      │
│  ─ POSTs results and heartbeats to report-scrape             │
└──────────────────────────────────────────────────────────────┘
```

Alerts leave through Resend (email) and the Telegram Bot API.

## Components

### Web app (`web/`)

React 18 + Vite. Visitors browse productions, pick one, and pay £2 for one month or £2 a month on auto-renew through Stripe Checkout. Subscribers manage a subscription from a tokenised link (`/manage?token=…`); there are no accounts. `/login` emails those links again.

`/monitor` is the operator dashboard. It signs in against `admin-auth` and then sends the admin credentials with every request: to `status-dashboard` for the health overview, and to the `admin-*` and `send-test-email` functions for its tools.

The per-production price is read from the `PRICE_PER_PRODUCTION_GBP_PENCE` secret (default 200, i.e. £2).

### Supabase (`supabase/`)

- **Database** — see [`DATABASE.md`](DATABASE.md). Row-level security is on for every table; the browser can read `productions` and nothing else.
- **Edge functions** (Deno). All are deployed with `verify_jwt = false`, so each one does its own authorisation:

  | Function | Who calls it | Authorised by | What it does |
  |---|---|---|---|
  | `create-checkout-session` | web app | public | Finds or creates the user, creates a Stripe Checkout Session, stores a `pending` subscription |
  | `stripe-webhook` | Stripe | Stripe signature | Activates, renews and cancels subscriptions; sends lifecycle email. De-duplicates on event id (`stripe_events`) |
  | `subscription-management` | manage page | management token | Shows a subscription, changes its notification channel, cancels it and applies the refund guarantee |
  | `request-manage-link` | `/login` | public | Emails the manage links for an address. Always answers "ok" so it does not reveal who is subscribed |
  | `telegram-webhook` | Telegram | optional secret header | Links a Telegram chat to a user from a one-time `/start` token |
  | `report-scrape` | the monitor | shared secret | Records heartbeats, updates each show's status, fans alerts out to subscribers |
  | `status-dashboard` | FAQ, `/monitor` | public for the check schedule; admin credentials for everything else | Health snapshot: per-show state, monitor heartbeat, database size, email usage, Stripe activity |
  | `admin-auth` | `/monitor` | admin credentials | Login check for the dashboard |
  | `admin-preview-cancel` | `/monitor` | admin credentials | Read-only preview of what cancelling a subscription would do |
  | `admin-test-fixture` | `/monitor` | admin credentials | Drives a hidden `test-fixture` production through the alert flow. See [`TESTING.md`](TESTING.md) |
  | `admin-create-production` | `/monitor` | admin credentials | Adds or updates a production and uploads its poster |
  | `send-test-email` | `/monitor` | admin credentials | Sends each email template with stub data |

Stripe test and live mode are chosen by the `STRIPE_SECRET_KEY` prefix; see [`STRIPE_MODES.md`](STRIPE_MODES.md).

### The monitor (`firefox-extension/`)

A WebExtension that runs in Firefox on an always-on machine. See [`firefox-extension/README.md`](https://github.com/BowerHarry/YellowSticker/blob/main/firefox-extension/README.md) for setup.

Each cycle (every 10 minutes by default, within configurable active hours) it:

1. reads the list of productions from Supabase with the publishable key, keeping those with an adapter, not paused, and inside their run dates;
2. checks today's performances of each for standing-ticket availability;
3. POSTs `{ kind: 'scrape', productionId, status, standCount, performanceCount, … }` to `report-scrape`, with its current schedule so the dashboard can tell "offline" from "outside active hours".

It also reports `boot`, and `stuck` / `resumed` when it has been unable to read availability for several cycles, so the operator gets an email.

### Notifications

When a report says `available`:

1. `report-scrape` sets `productions.last_standing_tickets_found_at` on every such cycle. The refund guarantee reads this.
2. On a flip from not-available to available it also sets `productions.last_availability_transition_at`. This marks the start of an availability event. A cycle that fails to read availability leaves the status as it was, so it does not start a new event.
3. The fan-out selects subscriptions that are `paid`, not past `subscription_end`, and whose `last_alerted_at` is empty or earlier than the event's start. Each gets an email, a Telegram message or both, according to its `notification_preference`; a Telegram-only subscriber with no linked chat is emailed instead. On success `last_alerted_at` is set and a `notification_logs` row is written.
4. On the flip, an operator copy also goes to `ALERT_EMAIL`.

Consequences:

- Each subscriber gets one alert per availability event, not one per cycle.
- Someone who subscribes while tickets are available is alerted on the next cycle.
- A send that fails is retried on the next cycle while tickets remain available.
- Alerts go out in batches of four, oldest subscriptions first, capped at 200 per report.

## Failure modes

- **Monitor offline** → no heartbeats. Within its active hours `/monitor` flags the monitor as unhealthy once no heartbeat has arrived for twice the poll interval; outside them it shows as paused. Nothing pushes this to the operator.
- **Monitor running but unable to read availability** → it reports `stuck` after several failed cycles and `report-scrape` emails the operator, at most once every three hours.
- **Resend or Telegram outage** → the report is still stored. Failed sends are retried each cycle while tickets remain available; if they sell out first, that alert is lost.
- **Supabase outage** → the monitor's POSTs fail; it logs and tries again next cycle.
- **Stripe redelivers an event** → acknowledged and skipped. If handling fails part-way, the event is released so the retry runs.
