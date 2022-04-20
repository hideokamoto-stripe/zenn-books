---
title: "use-shopping-cartで、カート機能を追加しよう"
---

このステップでは、複数の商品をまとめて注文できる「カート機能」を実装します。

## `<DebugCart>`コンポーネントで、カート機能のデバッグ準備をしよう

まず、カートの中身をデバッグするコンポーネントを配置します。

`pages/_app.js`を以下のように変更しましょう。

```diff jsx
import '../styles/globals.css'
import 'bootstrap/dist/css/bootstrap.min.css'
import { Navbar, Container } from 'react-bootstrap'
-import { CartProvider } from 'use-shopping-cart'
+import { CartProvider, DebugCart } from 'use-shopping-cart'

function MyApp({ Component, pageProps }) {
  return (
    <CartProvider
      mode="payment"
      cartMode="client-only"
      stripe={process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_API_KEY}
      currency="JPY"
      successUrl="http://localhost:3000/success"
      cancelUrl="http://localhost:3000"
    >
      <Navbar bg="dark" variant="dark">
        <Container>
          <Navbar.Brand href="#home">
            Hello EC
          </Navbar.Brand>
        </Container>
      </Navbar>
      <Component {...pageProps} />
+      <div style={{
+        width: '30%',
+        position: 'fixed',
+        bottom: 50,
+        right: 50,
+        overflowX: 'scroll',
+        padding: 20,
+        backgroundColor: '#fefefe',
+        boxShadow: '0 0 8px gray'
+      }}>
+       <DebugCart style={{}}/>
+      </div>
    </CartProvider>
  )
}

export default MyApp

```

これでページ下部にデバッグ用コンポーネントが表示されます。

![](https://storage.googleapis.com/zenn-user-upload/eb316ea76c4c-20220420.png)

### Tips: 開発環境でのみ表示する方法

このコンポーネントを利用しつつ、本番環境では非表示にしたい場合は、`process.env.NODE_ENV`が使えます。

三項演算子を使うことで、`production`では非表示にできます。

```jsx
      {process.env.NODE_ENV !== 'production' ? (
        <div style={{
          width: '30%',
          position: 'fixed',
          bottom: 50,
          right: 50,
          overflowX: 'scroll',
          padding: 20,
          backgroundColor: '#fefefe',
          boxShadow: '0 0 8px gray'
        }}>
        <DebugCart style={{}}/>
        </div>
       ) : null}
```

## `useShoppingCart`で、カート機能を実装

続いてカート機能を実装します。

### 商品追加ボタンを追加しよう

まずは商品を追加するボタンを追加しましょう。
`pages/index.js`を以下のように変更します。

```diff jsx
import Head from 'next/head'
import { Container, Row , Col, Image, Stack, Button } from "react-bootstrap"
+import { useShoppingCart } from 'use-shopping-cart'

// 中略
-  const { checkoutSingleItem } = useShoppingCart()
+  const { addItem } = useShoppingCart()
  return (
// 中略
                            <form action="/api/checkout_session" method="POST">
                              <input type='hidden' name='price' value={price.id}/>
                              <input type='hidden' name='quantity' value={1}/> 
                              <Button type='submit'>いますぐ注文する</Button>
                            </form>
                          </dd>
+                          <dd>
+                            <Button onClick={() => addItem({
+                              id: price.id,
+                              name: product.name,
+                              price: price.unit_amount,
+                              currency: price.currency,
+                              image: product.images[0],
+                            })}>カートに追加する</Button>
+                          </dd>
```

保存すると、「カートに追加する」ボタンが表示されます。

クリックすると、右下の`DebugCart`の金額や数量が変化します。

![](https://storage.googleapis.com/zenn-user-upload/69447af351e3-20220420.png)

### カートの中身を表示しよう

続いてカートの中身を表示するコンポーネントを作りましょう。

`components/cart.jsx`を新しく作成し、以下のコードを追加しましょう。

```jsx

import { Stack, Card, ButtonGroup, Button } from "react-bootstrap"
import { useShoppingCart } from 'use-shopping-cart'

export function CartDetail() {
    const { cartDetails } = useShoppingCart()
    return (
      <Stack gap={1}>
          {Object.entries(cartDetails).map(([priceId, detail]) => {
              return (
                <Card key={priceId}>
                    <Card.Body>
                        <Card.Title>{detail.name}</Card.Title>
                        <Card.Text>{detail.formattedPrice} * {detail.quantity} = {detail.formattedValue} {detail.currency}</Card.Text>
                        <ButtonGroup>
                            <Button variant="outline-danger">削除</Button>
                        </ButtonGroup>
                    </Card.Body>
                </Card>
              )
          })}
      </Stack>
    )
  }
```

続いて、`pages/index.js`を以下のように変更します。

```diff jsx
import { useShoppingCart } from 'use-shopping-cart'
+import { CartDetail } from '../components/cart'

export async function getStaticProps() {
// 中略


      <Container>
+        <Row>
+          <Col>
            <Stack gap={3}>
              {products.map(product => {
                // 中略
              })}
            </Stack>
+          </Col>
+          <Col md={4}>
+            <CartDetail />
+          </Col>
+        </Row>
      </Container>
```

変更を保存すると、ページ右上にカートの中身が表示されます。

![](https://storage.googleapis.com/zenn-user-upload/866a02e2aeff-20220420.png)

### カートの商品を削除できるようにしよう

追加した商品を削除するボタンも、Hookで簡単に実装できます。

`components/cart.jsx`を以下のように変更しましょう。

```diff jsx

import { Stack, Card, ButtonGroup, Button } from "react-bootstrap"
import { useShoppingCart } from 'use-shopping-cart'

export function CartDetail() {
-    const { cartDetails } = useShoppingCart()
+    const { cartDetails, removeItem } = useShoppingCart()
    return (
      <Stack gap={1}>
          {Object.entries(cartDetails).map(([priceId, detail]) => {
              return (
                <Card key={priceId}>
                    <Card.Body>
                        <Card.Title>{detail.name}</Card.Title>
                        <Card.Text>{detail.formattedPrice} * {detail.quantity} = {detail.formattedValue} {detail.currency}</Card.Text>
                        <ButtonGroup>
-                            <Button variant="outline-danger"}>削除</Button>
+                            <Button variant="outline-danger" onClick={() => removeItem(priceId)}>削除</Button>
                        </ButtonGroup>
                    </Card.Body>
                </Card>
              )
          })}
      </Stack>
    )
  }
```

[削除]ボタンに、`removeItem`関数を設定しました。

これでカート内の商品の[削除]ボタンをクリックすると、商品を削除できます。

## カートの合計金額を表示しよう

カート内の合計金額についても、ある程度まで`useShoppingCart`フックで計算できます。

`components/cart.jsx`を以下のように変更しましょう。


```diff jsx

import { Stack, Card, ButtonGroup, Button } from "react-bootstrap"
import { useShoppingCart } from 'use-shopping-cart'

export function CartDetail() {
-    const { cartDetails, removeItem } = useShoppingCart()
+    const { cartDetails, removeItem, formattedTotalPrice } = useShoppingCart()
    return (
      <Stack gap={1}>
          {Object.entries(cartDetails).map(([priceId, detail]) => {
              return (
                <Card key={priceId}>
                    <Card.Body>
                        <Card.Title>{detail.name}</Card.Title>
                        <Card.Text>{detail.formattedPrice} * {detail.quantity} = {detail.formattedValue} {detail.currency}</Card.Text>
                        <ButtonGroup>
                            <Button variant="outline-danger" onClick={() => removeItem(priceId)}>削除</Button>
                        </ButtonGroup>
                    </Card.Body>
                </Card>
              )
          })}
+          <Card>
+            <Card.Header>合計</Card.Header>
+            <Card.Body>
+              <Card.Title>{formattedTotalPrice}</Card.Title>
+              <ButtonGroup>
+                <Button variant='primary'>注文する</Button>
+                <Button variant="outline-danger">カートを空にする</Button>
+              </ButtonGroup>
+            </Card.Body>
+          </Card>
      </Stack>
    )
  }
```

これで合計金額を表示できるようになりました。

![](https://storage.googleapis.com/zenn-user-upload/82b3c8adce61-20220420.png)

## カートの中身を空にしよう

カートの中身を空にする方法も簡単です。
`components/cart.jsx`を以下のように変更しましょう。

```diff jsx

import { Stack, Card, ButtonGroup, Button } from "react-bootstrap"
import { useShoppingCart } from 'use-shopping-cart'

export function CartDetail() {
-    const { cartDetails, removeItem, formattedTotalPrice } = useShoppingCart()
+    const { cartDetails, removeItem, formattedTotalPrice, clearCart } = useShoppingCart()
    return (
      <Stack gap={1}>
          {Object.entries(cartDetails).map(([priceId, detail]) => {
              return (
                <Card key={priceId}>
                    <Card.Body>
                        <Card.Title>{detail.name}</Card.Title>
                        <Card.Text>{detail.formattedPrice} * {detail.quantity} = {detail.formattedValue} {detail.currency}</Card.Text>
                        <ButtonGroup>
                            <Button variant="outline-danger" onClick={() => removeItem(priceId)}>削除</Button>
                        </ButtonGroup>
                    </Card.Body>
                </Card>
              )
          })}
          <Card>
            <Card.Header>合計</Card.Header>
            <Card.Body>
              <Card.Title>{formattedTotalPrice}</Card.Title>
              <ButtonGroup>
                <Button variant='primary'>注文する</Button>
-                <Button variant="outline-danger">カートを空にする</Button>
+                <Button variant="outline-danger" onClick={() => clearCart()}>カートを空にする</Button>
              </ButtonGroup>
            </Card.Body>
          </Card>
      </Stack>
    )
  }
```

これで[カートを空にする]をクリックすることで、カートの中身を空にできます。
## 注文ページにカートの中身を反映させよう

最後にカートの商品を実際に注文できるように実装します。
### クライアント側だけで組み込みしてみよう

まずは手軽にできるクライアント側の実装を試しましょう。
`components/cart.jsx`を以下のように変更します。

```diff jsx

import { Stack, Card, ButtonGroup, Button } from "react-bootstrap"
import { useShoppingCart } from 'use-shopping-cart'

export function CartDetail() {
-    const { cartDetails, removeItem, formattedTotalPrice, clearCart } = useShoppingCart()
+    const { cartDetails, removeItem, formattedTotalPrice, clearCart, redirectToCheckout, cartCount } = useShoppingCart()
    return (
      <Stack gap={1}>
          {Object.entries(cartDetails).map(([priceId, detail]) => {
              return (
                <Card key={priceId}>
                    <Card.Body>
                        <Card.Title>{detail.name}</Card.Title>
                        <Card.Text>{detail.formattedPrice} * {detail.quantity} = {detail.formattedValue} {detail.currency}</Card.Text>
                        <ButtonGroup>
                            <Button variant="outline-danger" onClick={() => removeItem(priceId)}>削除</Button>
                        </ButtonGroup>
                    </Card.Body>
                </Card>
              )
          })}
          <Card>
            <Card.Header>合計</Card.Header>
            <Card.Body>
              <Card.Title>{formattedTotalPrice}</Card.Title>
              <ButtonGroup>
-                <Button variant='primary'>注文する</Button>
+                <Button
+                    variant='primary'
+                    disabled={cartCount < 1}
+                    onClick={async () => {
+                        try {
+                            const result = await redirectToCheckout()
+                            if (result.error) throw new Error(result.error)
+                        } catch (e) {
+                            window.alert(e.message);
+                        }
+                    }}
+                >
+                    注文する
+                </Button>
                <Button variant="outline-danger" onClick={() => clearCart()}>カートを空にする</Button>
              </ButtonGroup>
            </Card.Body>
          </Card>
      </Stack>
    )
  }
```

`useShoppingCart`の`redirectToCheckout`を利用して、Stripe Checkoutの決済ページに移動します。

![](https://storage.googleapis.com/zenn-user-upload/e4b8e72f89cb-20220420.png)

対応していない商品を選んだ場合などでは、エラーメッセージが表示されます。

![](https://storage.googleapis.com/zenn-user-upload/89e41364b603-20220420.png)


### APIで作るCheckoutセッションに、カートの中身を反映させよう

APIを利用する組み込みに変更しましょう。
`pages/api/checkout_session.js`を以下のように変更します。

```diff js
import Stripe from 'stripe'

export default async function handler(req, res) {
    if (req.method.toLocaleLowerCase() !== 'post') {
        return res.status(405).end()
    }
    try {
-        const { price, quantity } = req.body
+        const { price, quantity, items } = req.body
+        const lineItems = items ? items.map(item => ({
+            price: item.id,
+            quantity: item.quantity,
+            adjustable_quantity: {
+                enabled: true,
+            }
+        })): [{
+            price,
+            quantity,
+            adjustable_quantity: {
+                enabled: true,
+                minimum: 1,
+                maximum: 10,
+            }
+        }]
        const stripe = new Stripe(process.env.STRIPE_API_KEY, {
            apiVersion: '2020-08-27',
            maxNetworkRetries: 3,
        })
        const session = await stripe.checkout.sessions.create({
            mode: 'payment',
+            line_items: lineItems,
-            line_items: [{
-                price,
-                quantity,
-                // 個数の変更をサポートする
-                adjustable_quantity: {
-                    enabled: true,
-                    minimum: 1,
-                    maximum: 10,
-                }
-            }],
            success_url: 'http://localhost:3000/success',
            cancel_url: 'http://localhost:3000/'
        })
-        res.redirect(301, session.url)
+        if (!items) return res.redirect(301, session.url)
+        res.status(200).json({
+            url: session.url
+        })
    } catch (e) {
        console.log(e)
        res.status(e.statusCode || 500).json({
            message: e.message
        })
    }
}
  
```

APIリクエストのBodyで`item`が送られた場合、その内容でCheckoutのセッションを作成します。
意図せぬ値が送られてくるケースを防ぐため、`item`の中身を`map`で整形する処理も追加しました。
また、Ajaxを利用してJSONデータを送るため、APIで直接リダイレクトせずにURLを返す形に変更しましょう。


あとは`components/cart.js`を以下のように変更して、このAPIを呼び出しましょう。

```diff jsx
                <Button
                    variant='primary'
                    disabled={cartCount < 1}
                    onClick={async () => {
                        try {
+                            const session = await fetch('http://localhost:3000/api/checkout_session', {
+                                method: 'POST',
+                                headers: {
+                                    'Content-Type': 'application/json',
+                                },
+                                body: JSON.stringify({
+                                    items: Object.entries(cartDetails).map(([_id, detail]) => ({
+                                        id: detail.id,
+                                        quantity: detail.quantity,
+                                    }))
+                                })
+                            }).then(response => response.json())
+                            window.open(session.url)
-                            const result = await redirectToCheckout()
-                            if (result.error) throw new Error(result.error)
                        } catch (e) {
                            window.alert(e.message);
                        }
                    }}
                >
                    注文する
                </Button>
```

組み込みに成功すると、「パッケージ価格」にも対応した決済ページが新しいタブ・ウィンドウで表示されます。

![](https://storage.googleapis.com/zenn-user-upload/a7b07fb2cf1c-20220420.png)

## おさらい

- use-shopping-cartで、カート機能を簡単に実装できる
- カート画面のデバッグには、`DebugCart`コンポーネントを使おう
- `cartDetails`のデータを使ってCheckoutセッションを作成する

次のステップでは、注文のテストや、注文された商品情報をwebhookで受け取る方法を学びます。

## 「早く終わった」という方のための、もう１ステップ

`useShoppingCart`フックには、カート内商品の数量を変更する`incrementItem`や`decrementItem`など、さまざまな関数が用意されています。

カート内の商品数を増減させるボタンの追加に挑戦してみましょう。