# MONA Pay for Bubble

An API Connector reference and a small Node.js proxy that verifies MONA Pay webhooks before forwarding them to a Bubble backend workflow.

## Requirements

- Node.js 18 or later for `hmac-proxy.js` (no npm dependencies).
- An HTTPS reverse proxy in front of it; the proxy listens on `127.0.0.1` only.
- A Bubble app with the API Connector plugin and a backend workflow exposed as an API endpoint.

## API Connector

Bubble does not publish a stable import/export schema for the API Connector, so `api-connector.json` is a reference for configuring calls by hand, not a one-click import. It describes three calls against `https://api.monapay.vn`:

| Call | Method and path |
| --- | --- |
| Login | `POST /api/v1/client/login` |
| Generate VietQR | `POST /api/v1/acb/qr-payment/generate` |
| Recent transactions | `GET /api/v1/acb/virtual-account/transactions` |

1. Create an API named `MONA Pay` and add the calls with the method, path, headers and body from the JSON.
2. Mark the username, password, Client Secret, access token and ACB account parameters as private. Never put them in pages, custom states, URLs or front-end workflows.
3. In a backend workflow, call Login, read `data.access_token`, then call Generate VietQR. `amount` must be an integer in VND up to 1,000,000,000; `description` is at most 255 characters.
4. The response field `data.qr_data_url` is an EMVCo payload, not an image URL. Render it with a QR plugin you trust; do not send it to an uncontrolled third-party service.

## Receiving webhooks

Bubble workflows do not guarantee access to the raw HTTP body, which HMAC verification needs. Run `hmac-proxy.js` instead:

1. MONA Pay sends `POST https://proxy.example/webhooks/monapay` (JSON, `HMAC_SHA256`).
2. The proxy checks `X-Mona-Timestamp` and `X-Mona-Signature` on the raw bytes, with a 300-second window.
3. It rejects payloads that are not `income` or lack `transaction_code`, `description`, `account_number` or a numeric `amount`. The MONA Pay test event (`transaction_code: DUMMY123`) is acknowledged without forwarding.
4. It forwards the JSON to `BUBBLE_WORKFLOW_URL` with an `X-Mona-Proxy-Token` header (8-second timeout).
5. Your Bubble workflow checks the token, finds the order by the code in `description`, compares `amount`, and creates a Transaction record with a unique `transaction_code` before marking the order paid.

Bubble privacy rules do not replace signature checks, and the workflow still needs its own idempotency, logging and reconciliation.

## Configuration

See `.env.example`:

| Variable | Notes |
| --- | --- |
| `PORT` | Defaults to `8788` |
| `MONAPAY_WEBHOOK_SECRET` | The MONA Pay webhook HMAC secret |
| `BUBBLE_WORKFLOW_URL` | Bubble backend workflow URL; must use HTTPS |
| `BUBBLE_SHARED_TOKEN` | Separate random token checked by the workflow |

The proxy does not read `.env`; load the variables with your process manager.

## Usage

```bash
node hmac-proxy.js
```

## Development

```bash
npm test
find . -name '*.js' -print0 | xargs -0 -n1 node --check
```

Tests do not call the production API.

API documentation: https://monapay.vn/docs

**MONA Pay is part of MONA Cloud by The MONA Group.**
