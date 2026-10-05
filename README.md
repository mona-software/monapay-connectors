# MONA Pay platform connectors

Connectors that receive MONA Pay payment webhooks and mark the matching order as paid on an e-commerce platform.

Repository: https://github.com/mona-software/monapay-connectors

## Contents

| Path | What it does |
| --- | --- |
| [`core/`](core/README.md) | Node.js webhook server: verifies the HMAC signature, extracts the order ID, retries the platform call and drops duplicate transactions. |
| [`adapters/shopify`](adapters/shopify/README.md) | Marks a Shopify order paid with the Admin GraphQL `orderMarkAsPaid` mutation. |
| [`adapters/haravan`](adapters/haravan/README.md) | Creates a `Sale` or `Capture` transaction on a Haravan order. |
| [`adapters/sapo`](adapters/sapo/README.md) | Creates a `sale` or `capture` transaction on a Sapo order (Private App). |
| [`adapters/kiotviet`](adapters/kiotviet/README.md) | Records a `Transfer` payment against a KiotViet Retail invoice. |
| [`adapters/nhanh`](adapters/nhanh/README.md) | Writes the transfer amount and transaction code to a Nhanh.vn order (Open API v3). |
| [`adapters/pancake`](adapters/pancake/README.md) | Configurable HTTP call for Pancake POS; the real endpoint has not been verified yet. |
| [`adapters/woocommerce`](adapters/woocommerce/README.md) | Sets `set_paid: true` on a WooCommerce order through the REST API v3. |
| [`shopify-app/`](shopify-app/README.md) | Shopify app (OAuth, `orders/create` webhook) that creates a MONA Pay hosted checkout for manual-payment orders and marks them paid. |
| [`modules/opencart-monapay`](modules/opencart-monapay/README.md) | OpenCart 4.1 payment extension using MONA Pay hosted checkout. |
| [`modules/prestashop-monapay`](modules/prestashop-monapay/README.md) | PrestaShop 8 payment module using MONA Pay hosted checkout. |
| [`modules/magento2-monapay`](modules/magento2-monapay/README.md) | Magento 2 VietQR payment method (scaffold) with an HMAC webhook endpoint. |
| [`modules/bubble-monapay`](modules/bubble-monapay/README.md) | API Connector reference plus a webhook proxy that forwards verified events to a Bubble workflow. |
| [`modules/ghost-monapay`](modules/ghost-monapay/README.md) | Webhook bridge that adds a label to a Ghost member after a verified payment. |

## Requirements

- Node.js 18 or later for `core/`, `adapters/` and the Node modules; Node.js 22 for `shopify-app/`.
- No third-party npm dependencies.
- `core/` uses the MONA Pay Node SDK (`@monapay/node`), installed with `npm install` (see [`core/README.md`](core/README.md)).
- PHP 8.1+ for the OpenCart module; the PHP version required by your PrestaShop 8 or Magento 2 install for those modules.

## Status

Every adapter calls the endpoint described in the platform's public documentation, but each one must be tested against a real test shop before use:

- **Pancake POS**: the endpoint and payload are not verified; the adapter refuses to start until you configure them.
- **Haravan, Sapo, KiotViet, Nhanh.vn, WooCommerce, Shopify adapter**: need a test shop to confirm transaction kind, order ID format and side effects.
- **Magento 2**: scaffold; the QR payload is displayed as text until you add a QR renderer.
- `ORDER_ID_REGEX` must match the transfer memo (description) your shop puts in the QR.

## Development

Run the root test suite (core + Shopify app):

```bash
npm install
npm test
```

Each module under `modules/` and `shopify-app/` has its own test command; see its README.

API documentation: https://monapay.vn/docs

## License

No license has been chosen for this repository yet.

**MONA Pay is part of MONA Cloud by The MONA Group.**
