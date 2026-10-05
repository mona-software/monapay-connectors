# MONA Pay for Shopify

A dependency-free Node.js app that connects Shopify manual-payment orders to the MONA Pay hosted checkout and marks them paid when MONA Pay confirms the payment.

## How it works

1. Shopify sends `orders/create` for a `pending` order whose manual payment method name contains `MONA Pay`.
2. The app creates a MONA Pay checkout with order code `SP<order_number>`, saves the link as the order's Additional details entry `MONA Pay link` and adds the tag `monapay-pending`.
3. A Checkout UI extension reads `GET /api/pay-link/{orderId}` with a Shopify session token to show the payment button and QR on the thank-you page.
4. MONA Pay sends `CHECKOUT_PAID`. The app verifies the HMAC on the raw body, matches checkout, order code and amount, marks the order paid (GraphQL `orderMarkAsPaid`, with a REST transactions fallback) and changes the tag to `monapay-paid`.

## Requirements

- Node.js 22 or later. No `npm install` is needed; the app uses only Node built-ins.
- A Shopify Partner app with scopes `read_orders,write_orders`.
- A MONA Pay API key (Client ID and Client Secret) per merchant.

## Configuration

The app does not read `.env` itself. Load variables with `source` in development or systemd `EnvironmentFile` in production. See `.env.example`:

| Variable | Default | Notes |
| --- | --- | --- |
| `SHOPIFY_API_KEY` | | Partner app client ID |
| `SHOPIFY_API_SECRET` | | Partner app client secret |
| `SHOPIFY_APP_URL` | | Public HTTPS URL of the app |
| `SHOPIFY_SCOPES` | `read_orders,write_orders` | |
| `SHOPIFY_API_VERSION` | `2026-07` | |
| `APP_SECRET_KEY` | | At least 32 random characters, e.g. `openssl rand -base64 48` |
| `MONAPAY_API_BASE` | `https://api.monapay.vn` | |
| `HOST` | `127.0.0.1` | Non-loopback hosts need `ALLOW_PUBLIC_BIND=true` |
| `PORT` | `8793` | |
| `DATA_DIR` | `./data` | `.env.example` uses `/var/lib/monapay-shopify` |
| `OUTBOUND_TIMEOUT_MS` | `5000` | Max `30000` |

Keep `APP_SECRET_KEY` stable and backed up. Changing or losing it makes stored tokens and secrets undecryptable.

### Shopify app setup

1. Replace `REPLACE_WITH_SHOPIFY_API_KEY` in `shopify.app.toml` with your app's client ID, and update `application_url` and `redirect_urls` if you host the app elsewhere.
2. In the Shopify Dev Dashboard, set the App URL and the allowed redirect URL `<app-url>/auth/callback`.
3. Check that scopes are `read_orders,write_orders` and the webhook API version is `2026-07`.
4. Run `shopify app deploy` to publish the five webhook subscriptions in `shopify.app.toml`.
5. Install on a store with `<app-url>/auth?shop=<store>.myshopify.com`.

During OAuth the app also registers shop-specific `orders/create` and `app/uninstalled` webhooks. Webhook IDs and order IDs are deduplicated, so repeated deliveries do not create extra checkouts.

## Usage (merchant)

1. Install the app. Shopify asks for `read_orders,write_orders` and returns to the app's settings page.
2. In Shopify Admin, go to **Settings → Payments → Manual payment methods → Create custom payment method** and create a method whose name contains `MONA Pay`, for example *Chuyển khoản MONA Pay*.
3. In the MONA Pay dashboard, create an API key. Paste the Client ID and Client Secret into the settings page, click **Kiểm tra** (test), enable sandbox if you are testing, then click **Lưu cấu hình** (save). The settings page is in Vietnamese. The app creates or updates the MONA Pay webhook itself.
4. Place a VND order with that payment method. The order gets the `monapay-pending` tag and the `MONA Pay link` entry.
5. Pay and check that the order becomes `paid` with tag `monapay-paid`. For live use, turn off sandbox, save, and test a small real payment.

Do not use Shopify's **Send invoice** action for these orders; it is a separate payment flow.

## Endpoints

| Method | Path | Purpose |
| --- | --- | --- |
| `GET` | `/auth?shop=<shop>.myshopify.com` | Start OAuth (offline token) |
| `GET` | `/auth/callback` | Verify state and HMAC, exchange token, register webhooks |
| `GET` | `/settings?shop=...` | Merchant settings page; needs a session token or HMAC-signed cookie |
| `POST` | `/settings/test?shop=...` | Test MONA Pay credentials without saving |
| `POST` | `/settings/save?shop=...` | Save encrypted credentials and create or update the MONA Pay webhook |
| `POST` | `/webhooks/orders-create` | Shopify order created |
| `POST` | `/webhooks/monapay` | MONA Pay `CHECKOUT_PAID` |
| `POST` | `/webhooks/app-uninstalled` | Delete the shop's token and data |
| `POST` | `/webhooks/customers-data-request` | Compliance; the app stores no customer data |
| `POST` | `/webhooks/customers-redact` | Compliance; the app stores no customer data |
| `POST` | `/webhooks/shop-redact` | Compliance; delete all shop data |
| `GET` | `/api/pay-link/{orderId}` | Link and QR for the extension; needs a Shopify session token |
| `GET` | `/healthz` | Internal health check |

`/api/pay-link/{orderId}` accepts a numeric ID or a URL-encoded Shopify Order GID with `Authorization: Bearer <Shopify session token>`. It returns only checkouts belonging to the shop in the token's `dest` claim, and 404 until the checkout exists.

## Development

```bash
cp .env.example .env
# Fill in credentials and generate APP_SECRET_KEY before starting.
set -a
source .env
set +a
node server.js
```

Run tests:

```bash
npm test
# same as: node --test test/app.test.js test/crypto.test.js
```

Test files are listed explicitly because `node --test test/` does not scan the directory on Node 22.22.

### Sandbox end-to-end test

1. Enable sandbox on the settings page.
2. Create a VND order with the MONA Pay manual method. Check for `monapay-pending` and `MONA Pay link`.
3. Open the `https://pay.monapay.vn/c/<token>` link. It should say this is a test session; no real money moves.
4. In the MONA Pay dashboard, find the checkout by `SP<order_number>` to get its `SBX…` virtual account. Get a Bearer token with client credentials and send a simulated transaction:

```bash
curl -X POST https://api.monapay.vn/api/v1/sandbox/transactions \
  -H "Authorization: Bearer $MONA_TOKEN" \
  -H "X-Client-Secret: $MONA_CLIENT_SECRET" \
  -H 'Content-Type: application/json' \
  -d '{"virtual_account_number":"SBX...","amount":150000,"description":"SP1001","transaction_code":"SANDBOX-SP1001-01"}'
```

5. Check that MONA Pay sends `CHECKOUT_PAID`, the Shopify order becomes `paid` and the tag is `monapay-paid`.
6. Resend the same `transaction_code`; the app must return 200 without recording a second payment.

Also test a non-VND order, an order using another gateway, and an underpaid checkout. None of these should be marked paid.

## Deploy (Ubuntu, systemd, Nginx)

Sample files are in `deploy/`. They use the hostname `shopify.monapay.vn`; replace it with yours.

```bash
sudo useradd --system --home /var/lib/monapay-shopify --shell /usr/sbin/nologin monapay-shopify
sudo install -d -o monapay-shopify -g monapay-shopify -m 0700 /var/lib/monapay-shopify
sudo install -d -o root -g root -m 0755 /opt/monapay-shopify
sudo cp -a . /opt/monapay-shopify/
sudo install -o root -g root -m 0600 .env.example /etc/monapay-shopify.env
sudo install -o root -g root -m 0644 deploy/monapay-shopify.service /etc/systemd/system/
sudo install -o root -g root -m 0644 deploy/nginx-shopify.monapay.vn.conf /etc/nginx/sites-available/shopify.monapay.vn
sudo ln -s /etc/nginx/sites-available/shopify.monapay.vn /etc/nginx/sites-enabled/shopify.monapay.vn
sudo nginx -t
sudo systemctl daemon-reload
sudo systemctl enable --now monapay-shopify
```

Fill in real secrets in `/etc/monapay-shopify.env` before starting, and issue a TLS certificate before enabling the 443 server block. Nginx must pass the request body through unchanged; do not add a proxy or parser that re-serializes webhook bodies.

After deploying:

```bash
curl -fsS http://127.0.0.1:8793/healthz
curl -fsS https://<app-host>/healthz
systemctl status monapay-shopify
journalctl -u monapay-shopify -n 100 --no-pager
```

## Data and security

- The store is `$DATA_DIR/store.json` (file mode `0600`, directory mode `0700`), written via a temporary file and atomic rename, with a lock file against concurrent writers.
- Shopify access tokens, MONA Client Secrets and MONA webhook secrets are encrypted with AES-256-GCM using a key derived from `APP_SECRET_KEY`.
- Shopify and MONA Pay HMACs are computed on the raw request bytes. MONA Pay timestamps more than 300 seconds off are rejected.
- The app stores no customer names, emails, addresses or IDs, so `customers/data_request` and `customers/redact` only verify the request, write a minimal audit log and return 200.
- The JSON store supports a single instance. Move to a database with unique constraints before scaling horizontally.

## Thank-you extension

The backend is ready for a Checkout UI extension; see [`extensions/thank-you/README.md`](extensions/thank-you/README.md). The extension itself is not included yet.

## Known limitation

Webhook registration, order updates and the payment fallback use the REST Admin API (`webhooks.json`, `orders/{id}.json`, `orders/{id}/transactions.json`). Shopify treats REST Admin as legacy and requires new public apps to use GraphQL Admin, so these calls must be moved to GraphQL (or cleared with Shopify) before App Store submission.

References:

- https://shopify.dev/docs/apps/build/authentication-authorization/access-tokens/authorization-code-grant
- https://shopify.dev/docs/apps/build/webhooks/subscribe
- https://shopify.dev/docs/apps/build/compliance/privacy-law-compliance
- https://shopify.dev/docs/apps/build/authentication-authorization/session-tokens
- MONA Pay API: https://monapay.vn/docs

**MONA Pay is part of MONA Cloud by The MONA Group.**
