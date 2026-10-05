# Sapo adapter

Records a payment on a Sapo order by calling `POST /admin/orders/{id}/transactions.json` with Private App Basic authentication.

## Requirements

- A Sapo Private App with permission to create order transactions.

## Configuration

```dotenv
CONNECTOR_PLATFORM=sapo
SAPO_STORE_URL=https://your-shop.mysapo.net
SAPO_API_KEY=...
SAPO_API_SECRET=...
SAPO_TRANSACTION_KIND=sale
ORDER_ID_REGEX=MONA\s+SAPO\s+(?<orderId>\d+)
```

- `SAPO_STORE_URL` must use HTTPS.
- `SAPO_TRANSACTION_KIND` accepts `sale` (default, one-step payment) or `capture` (only when the order already has a matching authorization).

## Status

Not yet verified on a test shop. Confirm the domain, transaction permission and transaction kind on a manual-payment order before going live.

References:

- https://support.sapo.vn/transaction
- https://support.sapo.vn/gioi-thieu-order-api
- https://help.sapo.vn/ung-dung-rieng-private-apps

**MONA Pay is part of MONA Cloud by The MONA Group.**
