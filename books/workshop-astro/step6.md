---
title: "Webhook"
---

**src/pages/api/webhook.ts**

```ts
import { APIRoute } from "astro";
import Stripe from "stripe";

const STRIPE_SECRET_API_KEY = import.meta.env.STRIPE_SECRET_API_KEY;
const STRIPE_WEBHOOK_SECRET = import.meta.env.STRIPE_WEBHOOK_SECRET;

const stripe = new Stripe(STRIPE_SECRET_API_KEY, {
	apiVersion: "2022-11-15"
});

export const post: APIRoute = async (context) => {
    const { request } = context;
    const signature = request.headers.get("stripe-signature");
    try {
        const body = await request.arrayBuffer();
        const event = stripe.webhooks.constructEvent(
            Buffer.from(body),
            signature,
            STRIPE_WEBHOOK_SECRET
        );
        return {
            body: JSON.stringify({
                message: "ok",
            })
        };
      } catch (err) {
        const errorMessage = `⚠️  Webhook signature verification failed. ${err.message}`
        console.log(errorMessage);
        return new Response(errorMessage, {
            status: 400
        })
      }
}
```

## TODO
ユースケースとかを、cloudflareワークショップから引用してくる