---
title: "リダイレクト型の決済セッションを作成する方法"
---

2つ目の方法では、サーバー側のAPIが必要となります。

AstroをSSG（静的サイトジェネレーター）として運用したい場合は、別途APIサーバーを用意することも検討しましょう。

https://stripe.com/docs/checkout/quickstart


## AstroをSSRモードに変更する

TBD

## 決済ページにリダイレクトするAPIを作る

```ts:src/pages/api/checkout-session.ts
import { APIRoute } from "astro";

export const post: APIRoute = async () => {
    return {
        body: JSON.stringify({
            message: 'hello'
        })
    }
}
```

## フロントエンドから、注文セッションを開始する


Astro側では、`form`タグを利用して、クイックスタートが提供するPOSTのAPI（`/create-checkout-session`）に遷移する実装を追加します。

```js
---
import Layout from '../layouts/Layout.astro';
---

<Layout title="Welcome to Astro.">
	<main>
		<h1>Welcome to <span class="text-gradient">Astro</span></h1>
		<form action='/api/checkout-session' method='post'>
			<button type='submit'>Checkout</button>
		</form>
```

## API Key


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

## 決済ページにリダイレクトする実装を作る

```diff ts:src/pages/api/checkout-session.ts
import { APIRoute } from "astro";
+import Stripe from 'stripe';
+const STRIPE_SECRET_API_KEY = import.meta.env.STRIPE_SECRET_API_KEY;
+const stripe = new Stripe(STRIPE_SECRET_API_KEY, {
+	apiVersion: '2022-11-15'
+});

-export const post: APIRoute = async () => {
-    return {
-        body: JSON.stringify({
-            message: 'hello'
-        })
-    }
+export const post: APIRoute = async ({request}) => {
+    const origin = request.headers.get("Referer") || request.headers.get("Origin")
+    const product = await stripe.products.create({
+        name: 'Astro demo'
+    })
+    const session = await stripe.checkout.sessions.create({
+        success_url: `${origin}/thanks`,
+        cancel_url: origin,
+        line_items: [{
+            price_data: {
+                unit_amount: 1000,
+                currency: 'jpy',
+                recurring: {
+                    interval: 'month',
+                },
+                product: product.id
+            },
+            quantity: 1,
+            adjustable_quantity: {
+                enabled: true,
+                maximum: 10,
+                minimum: 1,
+            }
+        }],
+        mode: 'subscription',
+    })
+    return Response.redirect(session.url, 301)
}
```

これで`Checkout`ボタンをクリックすると、決済フォームへリダイレクトされるようになります。

![スクリーンショット 2023-02-07 13.11.49.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2366300/3bc37f04-0df0-5337-b157-422afc247ec2.png)

## ss
![](https://storage.googleapis.com/zenn-user-upload/03bfc8bd3bf1-20230418.png)

![](https://storage.googleapis.com/zenn-user-upload/1934d10a9372-20230418.png)


## Ref

https://docs.astro.build/ja/core-concepts/endpoints/


## TODO

SSRのAdaptorどうする？
Cloudflareにするか、Vercelあたりにするか、２・３個触れるか・・・