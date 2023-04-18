---
title: "Code&command snippets (Stripe Checkout)"
---


## Install Stripe SDK

```bash
% npm i stripe
```

```
info Direct dependencies
└─ stripe@12.1.1
info All dependencies
├─ call-bind@1.0.2
├─ has-symbols@1.0.3
├─ object-inspect@1.12.3
├─ qs@6.11.1
├─ side-channel@1.0.4
└─ stripe@12.1.1
✨  Done in 11.54s.
```

## Get API key

![](https://storage.googleapis.com/zenn-user-upload/d8630ed76b1f-20230418.png)
![](https://storage.googleapis.com/zenn-user-upload/95947d9a17eb-20230418.png)
![](https://storage.googleapis.com/zenn-user-upload/63c13d18c088-20230418.png)


### 余裕があれば制限付きキーの話


## Put API key

**`.env`**

```
PUBLIC_STRIPE_PUBLISHABLE_API_KEY=pk_test_xxx
STRIPE_SECRET_API_KEY=sk_test_xxx
```

## Test run

**src/pages/index.astro**

```diff
---
import Layout from '../layouts/Layout.astro';
import Card from '../components/Card.astro';
+import Stripe from 'stripe';
+const STRIPE_SECRET_API_KEY = import.meta.env.STRIPE_SECRET_API_KEY;
+const stripe = new Stripe(STRIPE_SECRET_API_KEY, {
+	apiVersion: '2022-11-15'
+});
---
```

```diff
---
import Layout from '../layouts/Layout.astro';
import Card from '../components/Card.astro';
import Stripe from 'stripe';
const STRIPE_SECRET_API_KEY = import.meta.env.STRIPE_SECRET_API_KEY;
const stripe = new Stripe(STRIPE_SECRET_API_KEY, {
	apiVersion: '2022-11-15'
});
+const customer = await stripe.customers.create({
+	name: 'First demo customer'
+})
---

<Layout title="Welcome to Astro!!.">
	<main>
-		<h1>Welcome to <span class="text-gradient">Astro</span></h1>
+		<h1>Welcome to <span class="text-gradient">Astro</span><br/>{ customer.name }</h1>
```

![](https://storage.googleapis.com/zenn-user-upload/0c621e497cab-20230418.png)
![](https://storage.googleapis.com/zenn-user-upload/61be3540af57-20230418.png)

## Checkout API

**src/pages/api/checkout.ts**
```ts
import { APIRoute } from "astro";

export const post: APIRoute = async ({request}) => {
    return {
        body: JSON.stringify({
            message: 'hello'
        })
    }
}
```

```
% curl -XPOST http://localhost:3000/api/checkout | jq .

{
  "message": "hello"
}
```

```ts
import { APIRoute } from "astro";
import Stripe from 'stripe';
const STRIPE_SECRET_API_KEY = import.meta.env.STRIPE_SECRET_API_KEY;
const stripe = new Stripe(STRIPE_SECRET_API_KEY, {
	apiVersion: '2022-11-15'
});

export const post: APIRoute = async ({request}) => {
    const origin = request.headers.get("Referer") || request.headers.get("Origin")
    const product = await stripe.products.create({
        name: 'Astro demo'
    })
    const session = await stripe.checkout.sessions.create({
        success_url: `${origin}/thanks`,
        cancel_url: origin,
        line_items: [{
            price_data: {
                unit_amount: 1000,
                currency: 'jpy',
                recurring: {
                    interval: 'month',
                },
                product: product.id
            },
            quantity: 1,
            adjustable_quantity: {
                enabled: true,
                maximum: 10,
                minimum: 1,
            }
        }],
        mode: 'subscription',
    })
    return Response.redirect(session.url, 301)
}
```

```diff
---
import Layout from '../layouts/Layout.astro';
import Card from '../components/Card.astro';
---

<Layout title="Welcome to Astro.">
	<main>
		<h1>Welcome to <span class="text-gradient">Astro</span></h1>
+		<form action="/api/checkout" method="POST">
+			<button type="submit">Subscribe</button>
+		</form>
```

![](https://storage.googleapis.com/zenn-user-upload/03bfc8bd3bf1-20230418.png)

![](https://storage.googleapis.com/zenn-user-upload/1934d10a9372-20230418.png)


## Ref

https://docs.astro.build/ja/core-concepts/endpoints/