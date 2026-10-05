# KiotViet adapter

Records a bank-transfer payment against a KiotViet Retail invoice by calling `POST https://public.kiotapi.com/payments`.

## Requirements

- KiotViet Public API client credentials (`client_credentials` grant, scope `PublicApi.Access`).
- The bank account ID in KiotViet that matches the `Transfer` payment method.

## Configuration

```dotenv
CONNECTOR_PLATFORM=kiotviet
KIOTVIET_CLIENT_ID=...
KIOTVIET_CLIENT_SECRET=...
KIOTVIET_RETAILER=your-retailer-code
KIOTVIET_ACCOUNT_ID=12345
ORDER_ID_REGEX=MONA\s+KIOTVIET\s+(?<orderId>\d+)
# Optional overrides
# KIOTVIET_TOKEN_URL=https://id.kiotviet.vn/connect/token
# KIOTVIET_API_BASE_URL=https://public.kiotapi.com
```

- The order ID must be the numeric KiotViet **invoice ID**, not an order code.
- The adapter caches the access token until 30 seconds before it expires.
- The request body is `{ amount, method: "Transfer", accountId, invoiceId }`; a response without `paymentId` is treated as a failure.

## Status

Not yet verified on a test shop. Only the Retail Public API is implemented; other KiotViet editions (FnB, Salon, Hotel) use different API bases. Create an unpaid invoice in a test store before going live.

References:

- https://www.kiotviet.vn/huong-dan-su-dung-kiotviet/retail-ket-noi-api/public-api/ (section 2.14.2, invoice payment)
- https://www.kiotviet.vn/huong-dan-su-dung-kiotviet/retail-ket-noi-api/ket-noi-api/

**MONA Pay is part of MONA Cloud by The MONA Group.**
