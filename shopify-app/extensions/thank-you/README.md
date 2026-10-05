# Thank-you Checkout UI extension

Integration notes for a Shopify Checkout UI extension that shows the MONA Pay payment link and QR on the thank-you and order status pages. The extension code is not in this repository yet; this file describes the backend contract it must use.

## Backend endpoint

```text
GET <app-url>/api/pay-link/{shopifyOrderId}
Authorization: Bearer <Shopify session token>
```

Response:

```json
{
  "checkout_url": "https://pay.monapay.vn/c/...",
  "qr_image_url": "https://api.monapay.vn/api/v1/checkouts/public/.../qr.png",
  "qr_data_url": "...",
  "amount": 150000,
  "currency": "VND",
  "order_code": "SP1001",
  "status": "pending",
  "expires_at": "2026-09-05T00:00:00Z"
}
```

The endpoint returns 404 until the `orders/create` webhook has created the checkout.

## Building the extension

Generate and link the extension with Shopify CLI against your real Partner app, so the target and Checkout UI package match the API version Shopify grants the app.

1. In `shopify-app/`, link the app to your Partner app and development store.
2. Use Shopify CLI to create a Checkout UI extension for the Thank you / Order status page.
3. Read the Order GID and a session token from the Checkout UI API. URL-encode the GID in the path.
4. Call the endpoint above. On 404, retry a limited number of times.
5. When `status` is `pending`, render the QR and a button that opens `checkout_url`. Do not recompute the amount or order code on the client.
6. When `status` is `paid`, show that payment was received and stop polling.
7. Allow network access to the app host if the extension manifest requires it, test accessibility and deploy with Shopify CLI.

Pseudocode:

```text
orderId = checkoutApi.order.id
token = await checkoutApi.sessionToken.get()
result = GET /api/pay-link/{encodeURIComponent(orderId)}
         Authorization: Bearer {token}

404 -> wait 2 seconds, retry up to 10 times
200 + pending -> show qr_image_url and a checkout_url button
200 + paid -> show "Payment received"
401 -> fetch a new session token once
```

Never put the MONA Client Secret, the Shopify offline token or the webhook secret in the extension.

References:

- https://shopify.dev/docs/api/checkout-ui-extensions/latest
- https://shopify.dev/docs/apps/build/checkout/thank-you-order-status
- https://shopify.dev/docs/apps/build/authentication-authorization/session-tokens

**MONA Pay is part of MONA Cloud by The MONA Group.**
