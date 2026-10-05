# MONA Pay for Magento 2

A Magento 2 payment method (scaffold) that creates a dynamic VietQR for VND orders after checkout and confirms payment through an HMAC-signed webhook at `POST /monapay/webhook/index`.

## Status

Scaffold, not yet tested on a live store. The QR payload is shown as text until you add a QR renderer (see Usage).

## Requirements

- Magento 2 with `Magento_Payment` and `Magento_Checkout`.
- A MONA Pay account (username, password, Client Secret) and the ACB QR parameters: `ownerNumber`, `ownerType` (`PER` or `ORG`), `merchantId`, `terminalId`, `virtualAccountPrefix`, `beneficiaryName`.
- Orders in VND.

## Install

1. Copy this directory to `app/code/Mona/MonaPay` (the directory name `magento2-monapay` will not work).
2. Run:

```bash
bin/magento module:enable Mona_MonaPay
bin/magento setup:upgrade
bin/magento cache:flush
```

## Configuration

1. Open **Stores → Configuration → Sales → Payment Methods → MONA Pay VietQR**.
2. Enter the API base URL (default `https://api.monapay.vn`), MONA Pay username, password, Client Secret, webhook HMAC secret and the six ACB QR values. Passwords and secrets are stored encrypted.
3. In MONA Pay, create a JSON webhook with `HMAC_SHA256` auth pointing to `https://shop.example/monapay/webhook/index`, using the same HMAC secret.
4. Enable the method only after testing a VND order on staging. It is disabled by default.

## Usage

- New orders are placed in `pending_payment`. On the success page the module logs in to MONA Pay, calls `POST /api/v1/acb/qr-payment/generate` with order code `DH<increment_id>` and memo `Thanh toan DH<increment_id>`, and stores the result in the payment's additional information.
- `qr_data_url` is an EMVCo payload, not an image URL. The success template prints it with the virtual account number and exposes it in a `data-monapay-qr` attribute. Render it as canvas or SVG with your theme's QR renderer before production; do not send it to a third-party QR service.
- The webhook verifies `HMAC-SHA256(secret, "<timestamp>.<raw_body>")` in constant time, rejects timestamps more than 300 seconds off, ignores duplicate `transaction_code` values, adds an order comment when the amount is short, and registers a capture only when the amount covers the grand total.
- Orders are matched only by `DH<increment_id>` in the webhook `description`.

## Development

```bash
find . -name '*.php' -print0 | xargs -0 -n1 php -l
php tests/hmac.php
```

Tests do not call the production API.

API documentation: https://monapay.vn/docs

## License

`composer.json` currently declares the license as `proprietary`.

**MONA Pay is part of MONA Cloud by The MONA Group.**
