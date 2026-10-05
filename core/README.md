# MONA Pay connector core

A Node.js server that receives MONA Pay webhooks at `POST /webhooks/monapay`, verifies the HMAC-SHA256 signature on the raw body, extracts the order ID and calls the configured platform adapter.

## Requirements

- Node.js 18 or later (uses the built-in `fetch`).
- Dependencies installed from the repository root: `npm install` (pulls the MONA Pay Node SDK, `@monapay/node`, which provides `verifyWebhook`).

## Configuration

The server does not read `.env` files. Pass variables through systemd, Docker or your shell. See `.env.example`.

| Variable | Required | Default | Notes |
| --- | --- | --- | --- |
| `CONNECTOR_PLATFORM` | yes | | `shopify`, `haravan`, `sapo`, `kiotviet`, `nhanh`, `pancake` or `woocommerce` |
| `MONA_WEBHOOK_SECRET` | yes | | Same value as the MONA Pay webhook `secret_key` |
| `HOST` | no | `127.0.0.1` | Non-loopback hosts are refused unless `ALLOW_PUBLIC_BIND=true` |
| `PORT` | no | `8787` | |
| `ORDER_ID_REGEX` | no | built-in patterns | Case-insensitive; see below |
| `PLATFORM_TIMEOUT_MS` | no | `2000` | Per attempt, capped at `2500` |
| `RETRY_DELAY_MS` | no | `250` | Base delay, max `500` |
| `BODY_LIMIT_BYTES` | no | `1048576` | Larger bodies get HTTP 413 |
| `IDEMPOTENCY_TTL_MS` | no | `86400000` | 24 hours |
| `IDEMPOTENCY_MAX_ENTRIES` | no | `10000` | |
| `LOG_LEVEL` | no | `info` | `debug`, `info`, `warn`, `error` |

Each adapter adds its own variables; see `../adapters/<platform>/README.md`.

### Order ID mapping

1. If the webhook payload has `orderId` or `order_id`, that value is used.
2. Otherwise the server matches `description` against `ORDER_ID_REGEX`. It takes the named group `orderId`, else the first group, else the whole match. Example: `MONA\s+SHOPIFY\s+(?<orderId>\d+)`.
3. Without `ORDER_ID_REGEX`, the defaults accept memos such as `MONA SHOPIFY 12345` or codes such as `DH10234`, `ORDER-10234`.

### Processing rules

- Invalid signatures get HTTP 401.
- Payloads with a `type` other than `income` are acknowledged with HTTP 202 and ignored.
- The adapter call is tried up to 3 times with 250 ms and 500 ms backoff. If it still fails the server returns HTTP 502 so MONA Pay can resend the webhook.
- Successful `transaction_code` values are kept in an in-memory cache for 24 hours; repeats return HTTP 200 with `duplicate: true`. The cache does not survive restarts and is not shared between replicas; for either case, replace it with a database table with a unique constraint on `transaction_code`.

## Usage

```bash
set -a
. /etc/monapay-connector.env
set +a
node core/server.js
curl --fail http://127.0.0.1:8787/healthz
```

`GET /healthz` returns `{"status":"ok"}`.

### systemd

Run as a non-root user bound to loopback:

```ini
[Unit]
Description=MONA Pay connector
After=network-online.target

[Service]
User=monapay-connector
Group=monapay-connector
WorkingDirectory=/opt/monapay-connectors
EnvironmentFile=/etc/monapay-connector.env
ExecStart=/usr/bin/node /opt/monapay-connectors/core/server.js
Restart=on-failure
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true

[Install]
WantedBy=multi-user.target
```

Keep the env file owned by `root:monapay-connector` with mode `640`. Expose only the webhook path through Nginx, which also terminates TLS:

```nginx
location = /webhooks/monapay {
    limit_except POST { deny all; }
    client_max_body_size 1m;
    proxy_pass http://127.0.0.1:8787;
    proxy_connect_timeout 2s;
    proxy_read_timeout 10s;
}
```

Do not expose `/healthz` publicly; monitor it from loopback.

### Docker

Build from the repository root:

```bash
docker build -f core/Dockerfile -t monapay-connector .
docker run --rm --network host --env-file /etc/monapay-connector.env monapay-connector
```

`--network host` lets the container keep binding `127.0.0.1` so Nginx on the host can reach it. The image runs as the `node` user.

## Development

```bash
npm test   # from the repository root; needs the SDK layout above
```

## License

No license has been chosen for this repository yet.

**MONA Pay is part of MONA Cloud by The MONA Group.**
