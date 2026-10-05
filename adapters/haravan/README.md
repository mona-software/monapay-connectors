# Haravan adapter

Records a payment on a Haravan order by calling `POST /com/orders/{order_id}/transactions.json`.

## Requirements

- A Haravan access token with the `com.write_orders` scope.

## Configuration

```dotenv
CONNECTOR_PLATFORM=haravan
HARAVAN_ACCESS_TOKEN=...
HARAVAN_TRANSACTION_KIND=Sale
ORDER_ID_REGEX=MONA\s+HARAVAN\s+(?<orderId>\d+)
# Optional, defaults to https://apis.haravan.com
# HARAVAN_API_BASE_URL=https://apis.haravan.com
```

- `HARAVAN_TRANSACTION_KIND` accepts `Sale` (default) or `Capture`. Use `Sale` for an external payment with no prior authorization. Use `Capture` only if your order flow already creates an authorization or pending transaction.
- The adapter does not change `financial_status` directly; Haravan documents that field as settable only when the order is created.
- A response without a `transaction`, or with a status other than `success`, is treated as a failure.

## Status

Not yet verified on a test shop. Confirm `Sale` versus `Capture` with a test order in your own store before going live.

References:

- https://docs.haravan.com/docs/omni-apis/transactions/
- https://docs.haravan.com/docs/omni-apis/orders/

**MONA Pay is part of MONA Cloud by The MONA Group.**
