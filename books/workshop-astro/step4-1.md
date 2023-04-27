---
title: "埋め込み型の決済フォームを、静的ページにて実装する"
---

## Payment Intent API

```typescript:src/pages/api/payment_intent.ts
import { APIRoute } from "astro";
import Stripe from 'stripe';
const STRIPE_SECRET_API_KEY = import.meta.env.STRIPE_SECRET_API_KEY;
const stripe = new Stripe(STRIPE_SECRET_API_KEY, {
	apiVersion: '2022-11-15'
});

export const post: APIRoute = async () => {
    const customer = await stripe.customers.create({
        name: 'Astro demo'
    })
    const intent = await stripe.paymentIntents.create({
      amount: 1000,
      currency: 'jpy',
      customer: customer.id,
      payment_method_types: ['card', 'customer_balance'],
      payment_method_data: {
        type: 'customer_balance'
      },
      payment_method_options: {
        customer_balance: {
          funding_type: 'bank_transfer',
          bank_transfer: {
            type: 'jp_bank_transfer'
          }
        }
      }
    })
    return {
        body: JSON.stringify({
            client_secret: intent.client_secret
        })
    }
}
```


### Bank transfer
```diff typescript

+    const customer = await stripe.customers.create({
+        name: 'Astro demo'
+    })
    const intent = await stripe.paymentIntents.create({
      amount: 1000,
      currency: 'jpy',
+      customer: customer.id,
+      payment_method_types: ['card', 'customer_balance'],
+      payment_method_data: {
+        type: 'customer_balance'
+      },
+      payment_method_options: {
+        customer_balance: {
+          funding_type: 'bank_transfer',
+          bank_transfer: {
+            type: 'jp_bank_transfer'
+          }
+        }
+      }
    })
```

## front end code

### SSRでPayment Intentを生成する処理を削除


```diff html:src/pages/index.astro
---
-import Stripe from 'stripe';
-const stripe = new Stripe(import.meta.env.STRIPE_SECRET_API_KEY);

-const paymentIntent = await stripe.paymentIntents.create({
-	amount: 100,
-	currency: "jpy",
-	automatic_payment_methods: {
-		enabled: true,
-	},
-});
-const clientSecret = paymentIntent.client_secret;
---
<form id='payment-element-form'>
-	<div id='payment-element' data-pi={clientSecret}></div>
+	<div id='payment-element'></div>
```

### Fetchでデータ取得

```diff html:src/pages/index.astro
<script>
import { loadStripe } from "@stripe/stripe-js";
const PUBLIC_STRIPE_PUBLISHABLE_API_KEY = import.meta.env.PUBLIC_STRIPE_PUBLISHABLE_API_KEY;
const stripe = await loadStripe(PUBLIC_STRIPE_PUBLISHABLE_API_KEY);

// Payment Intentを取得
-const elementPlaceholder = document.getElementById(this.elementId);
-const clientSecret = elementPlaceholder.dataset.pi;
+const response = await fetch('/api/payment_intent', {
+	method: 'POST'
+}).then(data => data.json());
+const clientSecret = response.client_secret;

// Elementの準備
const elements = stripe.elements({
	appearance: {
		theme: 'stripe',
	},
	clientSecret: response.client_secret,
});
```
## TODO
Screenshot
なぜ静的ページでやる？
静的に出力するための設定`predenrer=true`だったかの追加とテスト