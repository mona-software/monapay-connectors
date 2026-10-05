# Pancake POS adapter

A configurable HTTP call for marking a Pancake POS order as paid. The real endpoint has not been verified yet, so you must supply it.

## Status

The public Pancake POS documentation confirms an Open API and an order status flow, but the pages available did not specify the endpoint or payload for recording a payment. The adapter therefore ships with no built-in endpoint and refuses to start until every variable below is set with values taken from your Pancake POS or Partner account.

Still to confirm: endpoint, HTTP method, auth scheme, payment status field, whether the order ID is the internal ID or the order code, and the success response.

## Configuration

```dotenv
CONNECTOR_PLATFORM=pancake
# Replace every placeholder with values verified against the Pancake POS API.
PANCAKE_MARK_PAID_URL_TEMPLATE=https://<verified-host>/<verified-path>/{{orderId}}
PANCAKE_MARK_PAID_METHOD=<POST|PUT|PATCH>
PANCAKE_AUTH_HEADER=<verified-header-name>
PANCAKE_AUTH_VALUE=<verified-token-format>
PANCAKE_MARK_PAID_BODY_TEMPLATE={"<verified-field>":"<verified-value>","amount":"{{amount}}","reference":"{{transactionCode}}"}
ORDER_ID_REGEX=MONA\s+PANCAKE\s+(?<orderId>[A-Za-z0-9_-]+)
```

- Supported placeholders in the URL and body: `{{orderId}}`, `{{amount}}`, `{{transactionCode}}`. A JSON value that is exactly one placeholder keeps its type (for example a numeric amount).
- The URL must use HTTPS. The method must be `POST`, `PUT` or `PATCH`. The body template must be valid JSON.
- A response with `success: false` or `code: 0` is treated as a failure.
- The field names in the example are placeholders, not the Pancake schema.

References:

- https://docs.pancake.biz/pos/st-f13/st-p1?lang=vi (Open API overview)
- https://docs.pancake.biz/pos/api/ (Open API workspace)
- https://docs.pancake.biz/pos/st-f13/st-p3?lang=vi (order status flow)

**MONA Pay is part of MONA Cloud by The MONA Group.**
