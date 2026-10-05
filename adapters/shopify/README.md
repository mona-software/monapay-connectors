# Shopify adapter

Marks a Shopify order as paid by calling the Admin GraphQL mutation `orderMarkAsPaid(input: {id})`.

## Requirements

- An installed Shopify app with an Admin API access token and the `write_orders` scope (`read_orders` is useful for reconciliation).
- The order must use a manual payment method and still have a positive outstanding balance; otherwise Shopify returns `userErrors` and the connector reports a failure.

## Configuration

```dotenv
CONNECTOR_PLATFORM=shopify
SHOPIFY_SHOP=example.myshopify.com
SHOPIFY_ACCESS_TOKEN=shpat_...
SHOPIFY_API_VERSION=2026-07
ORDER_ID_REGEX=MONA\s+SHOPIFY\s+(?<orderId>\d+)
```

- `SHOPIFY_SHOP` must be a `*.myshopify.com` domain.
- `SHOPIFY_API_VERSION` must use the `YYYY-MM` form; pin a version Shopify still supports.
- The order ID can be numeric or a full `gid://shopify/Order/...` GID.

## Status

Not yet verified on a test shop. Before going live, confirm the mutation succeeds on a manual-payment order in your own store.

References:

- https://shopify.dev/docs/api/admin-graphql/latest/mutations/orderMarkAsPaid
- https://shopify.dev/docs/api/admin-graphql/latest/input-objects/OrderMarkAsPaidInput
- https://shopify.dev/docs/apps/build/authentication-authorization/access-tokens/generate-app-access-tokens-admin

**MONA Pay is part of MONA Cloud by The MONA Group.**
