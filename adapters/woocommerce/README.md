# WooCommerce adapter

Marks a WooCommerce order as paid by calling REST API v3 `PUT /wp-json/wc/v3/orders/{id}` with `{ "set_paid": true }`.

## Requirements

- A WooCommerce REST API key with Read/Write permission.
- HTTPS on the store, since the key is sent with Basic authentication.

## Configuration

```dotenv
CONNECTOR_PLATFORM=woocommerce
WOOCOMMERCE_STORE_URL=https://shop.example.com
WOOCOMMERCE_CONSUMER_KEY=ck_...
WOOCOMMERCE_CONSUMER_SECRET=cs_...
ORDER_ID_REGEX=MONA\s+WOOCOMMERCE\s+(?<orderId>\d+)
```

The order ID must be the internal WooCommerce order ID.

## Status

Not yet verified on a test shop. Run a staging order with your own plugins and gateways to check that the hooks fired by `set_paid` do not send unexpected emails or trigger other automation.

References:

- https://woocommerce.github.io/woocommerce-rest-api-docs/#update-an-order
- https://developer.woocommerce.com/docs/apis/rest-api/

**MONA Pay is part of MONA Cloud by The MONA Group.**
