---
title: "Stripe SDKで高機能な決済フォームを追加しよう"
---

続いてサーバー側でStripe SDKを利用して決済ページを表示させる方法を学びましょう。

## 決済ページに移動するAPIを用意しよう

サーバー側の処理も利用する場合、Next.jsでは新しくAPIを用意します。

`pages/api/checkout_session.js`を新しく作成しましょう。

作成したファイルに、以下のコードを追加します。

```js
import Stripe from 'stripe'

export default async function handler(req, res) {
  if (req.method.toLocaleLowerCase() !== 'post') {
      return res.status(405).end()
  }
  res.status(200).end()
}  
```

### Stripe SDKでCheckout Sessionを作成しよう

続いて、決済のためのセッションを作る実装を追加しましょう。

`pages/api/checkout_session.js`を以下のように変更します。

```diff js
import Stripe from 'stripe'

export default async function handler(req, res) {
    if (req.method.toLocaleLowerCase() !== 'post') {
        return res.status(405).end()
    }
-    res.status(200).end()
+    try {
+        const stripe = new Stripe(process.env.STRIPE_API_KEY, {
+            apiVersion: '2020-08-27',
+            maxNetworkRetries: 3,
+        })
+        const session = await stripe.checkout.sessions.create({
+            mode: 'payment',
+            line_items: [{
+                price_data: {
+                    unit_amount: 100,
+                    currency: 'jpy',
+                    product_data: {
+                        name: 'アドホックな商品'
+                    }
+                },
+                quantity: 1
+            }],
+            success_url: 'http://localhost:3000/success',
+            cancel_url: 'http://localhost:3000/'
+        })
+        res.status(200).json(session)
+    } catch (e) {
+        console.log(e)
+        res.status(e.statusCode || 500).json({
+            message: e.message
+        })
+    }
}
  
```

サーバー側の組み込みを利用すると、このサンプルのように「１度きりの商品・料金」を設定することもできます。
そのため、オーダーメイド製品やオプションの多い商品などを販売されている場合は、このように都度商品・料金データを作成する組み込みも検討できます。

https://qiita.com/hideokamoto/items/6bd31422e2ebb27d4eaa

### APIでリクエストBodyを受け取ろう

ECサイトへの組み込みでは、サイト側でユーザーが選択した商品でセッションを開始する必要があります。

そのため、APIのリクエストBodyから商品データを設定するように変更しましょう。

`pages/api/checkout_session.js`を以下のように変更します。

```diff js
import Stripe from 'stripe'

export default async function handler(req, res) {
    if (req.method.toLocaleLowerCase() !== 'post') {
        return res.status(405).end()
    }
    try {
+        const { price, quantity } = req.body
        const stripe = new Stripe(process.env.STRIPE_API_KEY, {
            apiVersion: '2020-08-27',
            maxNetworkRetries: 3,
        })
        const session = await stripe.checkout.sessions.create({
            mode: 'payment',
            line_items: [{
+                price,
+                quantity,
+                // 個数の変更をサポートする
+                adjustable_quantity: {
+                    enabled: true,
+                    minimum: 1,
+                    maximum: 10,
+                }
-                price_data: {
-                    unit_amount: 100,
-                    currency: 'jpy',
-                    product_data: {
-                        name: 'アドホックな商品'
-                    }
-                },
-                quantity: 1
            }],
            success_url: 'http://localhost:3000/success',
            cancel_url: 'http://localhost:3000/'
        })
        res.status(200).json(session)
    } catch (e) {
        console.log(e)
        res.status(e.statusCode || 500).json({
            message: e.message
        })
    }
}  
```

### 作成した決済ページにリダイレクトしよう

作成したセッションには、決済ページのURLが含まれています。

クライアント側で`window.open`を使って別タブで開くこともできますが、今回はシンプルにリダイレクトさせます。

`pages/api/checkout_session.js`を以下のように変更しましょう。

```diff js
+        res.redirect(301, session.url)
-        res.status(200).json(session)
```

これでサーバー側の組み込みが完了しました。

## APIをアプリに組み込もう

最後に注文ボタンを変更しましょう。

`pages/index.js`を以下のコードを参考に変更します。

```diff jsx
import Head from 'next/head'
import { Container, Row , Col, Image, Stack, Button } from "react-bootstrap"
-import { useShoppingCart } from 'use-shopping-cart'

// 中略

export default function Home({products}) {
-  const { checkoutSingleItem } = useShoppingCart()
  return (
// 中略
                          <dd>
-                            <Button>いますぐ注文する</Button>
+                            <form action="/api/checkout_session" method="POST">
+                              <input type='hidden' name='price' value={price.id}/>
+                              <input type='hidden' name='quantity' value={1}/> 
+                              <Button type='submit'>いますぐ注文する</Button>
+                            </form>

```

API側でリダイレクトまで行う実装に変更したため、一旦use-shopping-cartを外しています。
代わりに、`form`タグで遷移させるようにします。

ここまで組み込みが終わると、先ほどはエラーになった「パッケージ価格」の商品も注文できるようになります。

また、途中で追加した`adjustable_quantity`によって、商品の数を1-10で変更できるようになりました。

![](https://storage.googleapis.com/zenn-user-upload/c76c4f03f32b-20220420.png)


## おさらい

- Stripe Checkoutは、サーバー側でセッションを作成し、リダイレクトさせる
- 個数の変更やアドホックな価格の設定など、柔軟なカスタマイズが可能
- クライアント側でリダイレクトや`window.open`する実装にも変更できる

ここまでで、商品毎に注文を行う機能の実装に成功しました。
次のステップでは、複数の商品をまとめて注文できる「カート機能」を実装します。

## 「早く終わった」という方のための、もう１ステップ


Stripe Checkoutのセッション作成APIには、さまざまなオプションが用意されています。

ドキュメントを参考に、`expires_at`などの未紹介パラメーターについても試してみましょう。

https://stripe.com/docs/api/checkout/sessions/create
