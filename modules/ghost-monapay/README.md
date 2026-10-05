# MONA Pay for Ghost

A Node.js webhook bridge that verifies a MONA Pay payment and adds a label to the matching Ghost member through the Ghost Admin API.

## Scope

Ghost has no payment-provider plugin API, so this module cannot replace the Memberships/Portal checkout. It provides:

- A manual flow: a Ghost page that explains the bank transfer or VietQR payment.
- A bridge that adds the label `MONA Pay paid` (configurable) to a member after a verified payment.

The label is an admin marker only. It does not make the member a native paid subscriber, create a Stripe subscription, unlock Portal paid tiers or handle renewals and cancellations. If your content uses `visibility: paid`, you need a separate entitlement workflow checked against your Ghost version.

## Requirements

- Node.js 18 or later (no npm dependencies).
- A Ghost Admin API key (`id:hexsecret`).
- An HTTPS reverse proxy (Nginx, Caddy) in front of the bridge; it listens on `127.0.0.1` only.

## Configuration

See `.env.example`:

| Variable | Default | Notes |
| --- | --- | --- |
| `PORT` | `8787` | |
| `MONAPAY_WEBHOOK_SECRET` | | Required |
| `GHOST_URL` | | Required, e.g. `https://your-publication.example` |
| `GHOST_ADMIN_API_KEY` | | Required, `id:hexsecret` |
| `GHOST_MEMBER_MAP_FILE` | `./members.json` | Payment reference to member mapping |
| `MONAPAY_STATE_FILE` | `./data/processed.json` | Ledger of processed `transaction_code` values |
| `GHOST_PAID_LABEL` | `MONA Pay paid` | |

The bridge does not read `.env`; load the variables with your process manager. Do not commit `.env`, `members.json` or `data/`.

## Usage

1. Create a Ghost page explaining the bank transfer and collecting the member's email.
2. From your backend, create a MONA Pay QR whose `description` contains `MEMBER <reference>`, for example `Thanh toan MEMBER customer-0001`. Keep personal data out of the reference.
3. After confirming the member, add the reference to `members.json` with the Ghost member UUID and `expectedAmount` (see `members.example.json`). Payments below `expectedAmount` are rejected.
4. Point a MONA Pay webhook (JSON, `HMAC_SHA256`) at `https://bridge.example/webhooks/monapay`.
5. Start the bridge:

```bash
node server.js
```

For each webhook the bridge checks the signature on the raw body with a 300-second timestamp window, looks up the reference, loads the member, keeps existing labels, adds the paid label, and records the `transaction_code` in the ledger file (mode `0600`). The MONA Pay test event (`transaction_code: DUMMY123`) is acknowledged without changes.

The file ledger suits a single process. For multiple replicas, use a database with a unique `transaction_code` column. Keep the server clock in sync.

## Development

```bash
npm test
find . -name '*.js' -print0 | xargs -0 -n1 node --check
```

Tests do not call the production API.

API documentation: https://monapay.vn/docs

**MONA Pay is part of MONA Cloud by The MONA Group.**
