# MONA Pay for OpenCart 4.1

An OpenCart 4.1 payment extension that sends VND orders to the MONA Pay hosted checkout and confirms payment through an HMAC-signed webhook.

## Requirements

- OpenCart `4.1.0.4` (the tested target), PHP 8.1 or later.
- A MONA Pay API key (Client ID and Client Secret), a webhook HMAC secret and the return signature secret.
- Store currency VND. The method is hidden for other currencies, and order totals must be between 1,000 and 1,000,000,000 VND.

## Install

Copy into the web root:

```bash
cp -a upload/. /path/to/opencart/
```

The admin controller then lives at `extension/monapay/admin/controller/payment/monapay.php`.

To install through the admin Installer instead, build an `.ocmod.zip` whose root contains `install.json` plus the `admin/`, `catalog/` and `system/` folders from `upload/extension/monapay/` (do not include the `upload/extension/monapay` path itself). OpenCart derives the extension namespace from the zip file name, so name it `monapay.ocmod.zip`.

Then open **Extensions → Extensions → Payments**, install **MONA Pay** and edit it. Installing creates two tables: `monapay_checkout` (order to checkout mapping) and `monapay_transaction` (processed `transaction_code` ledger).

## Configuration

| Field | Value |
| --- | --- |
| API base URL | `https://api.monapay.vn` (HTTPS required) |
| Client ID / Client Secret | Your MONA Pay API key |
| Webhook HMAC secret | Secret of the JSON + `HMAC_SHA256` webhook |
| Return signature secret | From MONA Pay **Settings → Payment page**; not the webhook secret |
| Payment mode | `redirect` (the only supported mode) |
| Sandbox | Adds `"sandbox": true` to new checkouts; no real money moves |
| Paid order status | OpenCart status set after full payment |

Register this webhook URL in MONA Pay:

```text
https://shop.example/index.php?route=extension/monapay/payment/monapay.webhook
```

## Usage

1. On order confirmation the extension gets an OAuth client-credentials token, calls `POST /api/v1/checkouts` with order code `DH<order_id>` (idempotency key `opencart-<order_id>`, 15-minute expiry) and redirects the customer to the returned `checkout_url`.
2. The webhook verifies `X-Mona-Signature = sha256=HMAC_SHA256(secret, "<timestamp>.<raw_body>")`, rejects timestamps more than 300 seconds off, and handles `CHECKOUT_PAID` (matching checkout ID and order code) and `TRANSACTION_IN` (matching `DH<order_id>` in the description).
3. Duplicate `transaction_code` values are ignored; underpaid amounts do not change the order.
4. The return URL (`extension/monapay/payment/monapay.callback`) is added automatically. It verifies the redirect signature, then calls `GET /api/v1/checkouts/{id}` server-side before updating the order. The webhook remains the primary confirmation.

## Development

```bash
find upload tests -name '*.php' -print0 | xargs -0 -n1 php -l
php tests/hmac.php
```

API documentation: https://monapay.vn/docs

**MONA Pay is part of MONA Cloud by The MONA Group.**
