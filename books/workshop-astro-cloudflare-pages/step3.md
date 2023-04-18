---
title: "Code&command snippets (Elements)"
---

## Payment Intent API

**src/pages/api/payment_intent.ts**
```
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
```diff

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

## Elements

```bash
% npm i @stripe/stripe-js
```

**src/pages/index.astro**
HTML
```
<form id='payment-element-form'>
	<div id='payment-element'></div>
	<button type="submit">Order</button>
</form>
```

JS
```
<script>
import { loadStripe } from "@stripe/stripe-js";
const PUBLIC_STRIPE_PUBLISHABLE_API_KEY = import.meta.env.PUBLIC_STRIPE_PUBLISHABLE_API_KEY;
const stripe = await loadStripe(PUBLIC_STRIPE_PUBLISHABLE_API_KEY);

// Payment Intentを取得
const response = await fetch('/api/payment_intent', {
	method: 'POST'
}).then(data => data.json());

// Elementの準備
const elements = stripe.elements({
	appearance: {
		theme: 'stripe',
	},
	clientSecret: response.client_secret,
});
const paymentElement = elements.create('payment', {
	layout: 'tabs',
});
const elementPlaceholder = document.getElementById('payment-element');
paymentElement.mount(elementPlaceholder)

// Submit処理
document.getElementById('payment-element-form')
.addEventListener('submit', async (e) => {
	e.preventDefault();
	if (!stripe || !elements) return;
	const { error, paymentIntent } = await stripe.confirmPayment({
		elements,
		redirect: 'if_required'
	});
	if (error) {
		alert(`${error.code}: ${error.message}`);
	} else {
		alert("done");
	}
});
</script>
```