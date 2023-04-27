---
title: "埋め込み型の決済フォームを実装する方法"
---

3つ目の方法もまた、サーバー側のAPIが必要となります。

AstroをSSG（静的サイトジェネレーター）として運用したい場合は、別途APIサーバーを用意することも検討しましょう。

https://stripe.com/docs/payments/quickstart

## Stripe.jsライブラリを追加する

決済フォームを埋め込むためには、フロンドエンド用のSDKも追加しましょう。

```bash
% npm i  @stripe/stripe-js
```

## AstroをSSRモードに変更する

TBD

## フォームを配置するHTML要素を用意する

```html:src/pages/index.astro
---
const clientSecret = 'dummy'
---
<code>{clientSecret}</code>
<form id='payment-element-form'>
	<div id='payment-element' data-pi={clientSecret}></div>
	<button type="submit">Order</button>
</form>
```

## Payment Intentをサーバー側で作成する
Astro IslandでSSRとして

```diff html:src/pages/index.astro
---
import Stripe from 'stripe';
const stripe = new Stripe(import.meta.env.STRIPE_SECRET_API_KEY);

const paymentIntent = await stripe.paymentIntents.create({
	amount: 100,
	currency: "jpy",
	automatic_payment_methods: {
		enabled: true,
	},
});
-const clientSecret = 'dummy'
+const clientSecret = paymentIntent.client_secret;
---

```


## Stripe.jsでフォームを埋め込み

```diff html:src/pages/index.astro

-<code>{clientSecret}</code>
<form id='payment-element-form'>
	<div id='payment-element' data-pi={clientSecret}></div>
	<button type="submit">Order</button>
</form>
+<script>
+import { loadStripe } from "@stripe/stripe-js";
+const PUBLIC_STRIPE_PUBLISHABLE_API_KEY = import.meta.env.PUBLIC_STRIPE_PUBLISHABLE_API_KEY;
+const stripe = await loadStripe(PUBLIC_STRIPE_PUBLISHABLE_API_KEY);

+// Payment Intentを取得
+const elementPlaceholder = document.getElementById(this.elementId);
+const clientSecret = elementPlaceholder.dataset.pi;

+// Elementの準備
+const elements = stripe.elements({
+	appearance: {
+		theme: 'stripe',
+	},
+	clientSecret: response.client_secret,
+});
+const paymentElement = elements.create('payment', {
+	layout: 'tabs',
+});
+const elementPlaceholder = document.getElementById('payment-element');
+paymentElement.mount(elementPlaceholder)

+</script>
```

## 決済処理を完了する

```diff html:src/pages/index.astro
const elementPlaceholder = document.getElementById('payment-element');
paymentElement.mount(elementPlaceholder)

+// Submit処理
+document.getElementById('payment-element-form')
+.addEventListener('submit', async (e) => {
+	e.preventDefault();
+	if (!stripe || !elements) return;
+	const { error, paymentIntent } = await stripe.confirmPayment({
+		elements,
+		redirect: 'if_required'
+	});
+	if (error) {
+		alert(`${error.code}: ${error.message}`);
+	} else {
+		alert("done");
+	}
+});
</script>
```


### [Advanced]銀行振込をサポートする場合

```diff html:src/pages/index.astro
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

## TODO
Screenshot