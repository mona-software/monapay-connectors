# Nhanh.vn adapter

Writes the transfer amount and MONA Pay transaction code to a Nhanh.vn order through Open API v3 `POST /v3.0/order/edit`.

## Requirements

- A Nhanh.vn Open API v3 app ID, business ID and access token. Per Nhanh.vn docs, the token is valid for one year and has no automatic refresh.

## Configuration

```dotenv
CONNECTOR_PLATFORM=nhanh
NHANH_APP_ID=123
NHANH_BUSINESS_ID=456
NHANH_ACCESS_TOKEN=...
NHANH_TRANSFER_ACCOUNT_ID=789
# Set only after checking your business's order status IDs:
# NHANH_PAID_STATUS_ID=...
ORDER_ID_REGEX=MONA\s+NHANH\s+(?<orderId>\d+)
# Optional, defaults to https://pos.open.nhanh.vn
# NHANH_API_BASE_URL=https://pos.open.nhanh.vn
```

- The request body is `{ info: { id }, payment: { transferAmount, code, transferAccountId? } }`, sent as raw JSON with the token in the `Authorization` header.
- By default the adapter only records the payment. Set `NHANH_PAID_STATUS_ID` to also change `info.status`; take the ID from your own account's status list.
- A response with `code` other than `1` is treated as a failure.

## Status

Not yet verified on a test shop. Check whether your workflow needs a status change before going live.

References:

- https://apidocs.nhanh.vn/v3/order/edit
- https://apidocs.nhanh.vn/v3

**MONA Pay is part of MONA Cloud by The MONA Group.**
