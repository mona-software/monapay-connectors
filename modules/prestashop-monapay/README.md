# MONA Pay for PrestaShop 8

A PrestaShop 8.x payment module that creates a pending VND order, sends the customer to the MONA Pay hosted checkout and confirms payment through an HMAC-signed webhook.

## Requirements

- PrestaShop 8.0 to 8.x.
- A MONA Pay API key (Client ID and Client Secret), a webhook HMAC secret and the return signature secret.

## Install

The module directory inside PrestaShop must be named `monapay`:

```bash
cp -a prestashop-monapay /var/www/html/modules/monapay
cd /var/www/html
php bin/console prestashop:module install monapay --no-interaction
```

Installing adds the order state **Awaiting MONA Pay payment** (Vietnamese: *Chờ thanh toán MONA Pay*) and three tables:

- `ps_monapay_checkout`: order to hosted checkout mapping and status.
- `ps_monapay_transaction`: processed `transaction_code` ledger.
- `ps_monapay_token`: OAuth access token cache, refreshed 60 seconds before expiry.

On uninstall the token table and all configuration values (including secrets) are deleted; the checkout and transaction tables are kept for reconciliation.

## Configuration

Open **Module Manager → MONA Pay → Configure**:

| Field | Value |
| --- | --- |
| Base URL | `https://api.monapay.vn` (default) |
| Client ID / Client Secret | Your MONA Pay API key |
| Webhook Secret | Secret of the HMAC webhook |
| Return Signature Secret | From MONA Pay **Settings → Payment page**; not the webhook secret |
| Sandbox | On by default after install; adds `"sandbox": true` to new checkouts |

Secret fields left empty on save keep their current value.

Register this webhook in MONA Pay:

```text
https://your-domain/module/monapay/webhook
```

- Method: `POST`
- Payload: `application/json`
- Auth: `HMAC_SHA256`
- Event: `CHECKOUT_PAID`

The return URL `https://your-domain/module/monapay/return` is sent automatically when a checkout is created.

## Usage

1. `hookPaymentOptions()` adds the payment option (button label: *Thanh toán qua MONA Pay*).
2. `controllers/front/validation.php` creates the order in the pending state, gets an OAuth client-credentials token, calls `POST /api/v1/checkouts` with order code `DH<id_order>` and an `Idempotency-Key`, and redirects to `checkout_url`.
3. MONA Pay calls `/module/monapay/webhook`. The module verifies HMAC-SHA256 on the raw body with a 300-second timestamp window and processes only `CHECKOUT_PAID` events that match the checkout, order code and amount.
4. The module records `transaction_code` (primary key, so duplicates are ignored) and moves the order to **Payment accepted**.
5. `/module/monapay/return` verifies the redirect signature, calls `GET /api/v1/checkouts/{id}` and confirms the order only if the server-side status is `paid`. The browser query string alone never confirms an order.

## Development

```bash
find . -name '*.php' -print0 | xargs -0 -n1 php -l
php tests/hmac.php
```

Then test on staging: guest checkout, sandbox full payment, underpayment, a repeated `transaction_code`, a return with a bad signature, and a webhook older than 300 seconds.

API documentation: https://monapay.vn/docs

**MONA Pay is part of MONA Cloud by The MONA Group.**
