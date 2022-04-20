---
title: "use-shopping-cartで簡単な決済フォームを追加しよう"
---

ここからは、表示した商品の注文ボタンの実装を行います。

## Stripe Checkoutのクライアント側組み込みを有効化しよう

このステップでは、まず簡単な組み込み方法として、「クライアント側のみの組み込み」の準備を行います。

フロントエンド側のコードだけで実装できる手軽さがある反面、カード決済以外の決済方法やクーポンなどいくつかの機能が利用できなくなる制限もあります。

https://stripe.com/docs/payments/checkout/client

Stripeダッシュボード右上の[歯車アイコン]をクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/5cccfcf02e87-20220420.png)

設定ページに移動しますので、[Payments]の[CheckoutとPayment Links]をクリックします。

![](https://storage.googleapis.com/zenn-user-upload/cbd172b602ca-20220420.png)

ページをスクロールすると、[次のステップ]セクションに[Checkoutクライアント側の組み込み]があります。
[クライアント側のみの組み込みを有効にする]ボタンをクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/adb42fbd47b9-20220420.png)

注意画面が表示されますので、[許可]をクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/d0532e906f2c-20220420.png)

設定が完了すると、組み込み画面の表示が変わります。
設定については、本番環境で利用するための設定が中心ですので、今回は操作しません。

![](https://storage.googleapis.com/zenn-user-upload/81ffc765f70e-20220420.png)

## use-shopping-cartをインストールしよう

use-shopping-cartはReactアプリで、Stripeのカート・注文機能を利用できるようにするOSSライブラリです。

https://useshoppingcart.com/

npmでインストールすればすぐに利用できますので、さっそく`npm install use-shopping-cart`を実行しましょう。

```bash
npm install use-shopping-cart

...
✨  Done in 5.17s.
```

`package.json`の`dependencies`に`use-shopping-cart`が追加されていればOKです。

```diff json
{
  "name": "nextjs-stripe-ec",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "bootstrap": "^5.1.3",
    "next": "12.1.5",
    "react": "18.0.0",
    "react-bootstrap": "^2.2.3",
    "react-dom": "18.0.0",
    "stripe": "^8.218.0",
+    "use-shopping-cart": "^3.1.1"
```

## `CartProvider`を配置してセットアップしよう

use-shopping-cartはReact Hookを提供するライブラリです。
内部的にReduxを利用しているため、Hookを利用するにはProviderを配置する必要があります。

`pages/_app.js`を開き、以下のように変更しましょう。

```diff jsx
import '../styles/globals.css'
import 'bootstrap/dist/css/bootstrap.min.css'
import { Navbar, Container } from 'react-bootstrap'
+import { CartProvider } from 'use-shopping-cart'

function MyApp({ Component, pageProps }) {
  return (
+    <CartProvider
+      mode="payment"
+      cartMode="client-only"
+      stripe={process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_API_KEY}
+      currency="JPY"
+      successUrl="http://localhost:3000/success"
+      cancelUrl="http://localhost:3000"
+    >
-    <>
      <Navbar bg="dark" variant="dark">
        <Container>
          <Navbar.Brand href="#home">
            Hello EC
          </Navbar.Brand>
        </Container>
      </Navbar>
      <Component {...pageProps} />
+    </CartProvider>
-    </>
  )
}

export default MyApp
```

`CartProvider`では、Stripe Checkoutの組み込みに必要なパラメータを定義します。
`stripe`で公開可能キーを設定する必要がありますので、環境変数から設定しましょう。
なお、`NEXT_PUBLIC_`か**始まらない**環境変数はフロントエンド側で利用できませんのでご注意ください。

## `useShoppingCart`hookで、「いますぐ注文」ボタンを実装しよう

続いて注文ボタンに動きを追加しましょう。

`pages/index.js`を以下のように変更します。

```diff jsx
import Head from 'next/head'
import { Container, Row , Col, Image, Stack, Button } from "react-bootstrap"
+import { useShoppingCart } from 'use-shopping-cart'

// 中略

export default function Home({products}) {
+  const { checkoutSingleItem } = useShoppingCart()
  return (
// 中略
                          <dd>
+                            <Button onClick={() => checkoutSingleItem({ price: price.id })}>いますぐ注文する</Button>
-                            <Button>注文する</Button>


```

``useShoppingCart`を利用して、`checkoutSingleItem`関数を取得しています。
この関数にStripeの料金IDを渡すことで、StripeのCheckoutページに移動できます。

変更を保存し、登録した商品の「いますぐ注文する」ボタンをクリックしてみましょう。

画像のように、注文ページに移動すれば成功です。

![](https://storage.googleapis.com/zenn-user-upload/46b288d0256c-20220420.png)

なお、「パッケージ価格」で登録した商品は、「クライアント側の組み込み」では利用できませんのでご注意ください。


## おさらい

- Stripe Checkoutは、クライアント側だけで決済フォームの設定ができる
- use-shopping-cartで、Stripeまわりの操作ができるReact Hookが使える
- 送料やクーポン入力の設定など、より多くの機能を使うにはサーバー側の処理が必要

続いて、サーバー側でStripe SDKも利用して組み込む方法を学びましょう。

##　「早く終わった」という方のための、もう１ステップ

use-shopping-cartには、通貨や言語に応じた価格テキストを出す関数など、ヘルパー関数が用意されています。

https://useshoppingcart.com/docs/usage/helpers/format-currency-string

ヘルパー関数を利用して、サイトの価格表示を変更してみましょう。