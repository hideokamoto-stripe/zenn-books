---
title: "Stripeに登録した商品を、Next.jsで取得・表示しよう"
---

このステップでは、Stripeに登録した商品データをサイトに表示させる方法を体験します。

## `pages/index.js`を商品一覧ページに書き換えよう

シンプルな例として、サイトのTOPページに商品一覧を表示させましょう。

`pages/index.js`のコードを削除し、以下のコードを入力します。

```js
import Head from 'next/head'
import { useState, useEffect } from 'react'

export default function Home() {
  const [products, setProducts] = useState([])
  useEffect(() => {
    fetch('http://localhost:3000/api/products')
      .then(response => response.json())
      .then(data => setProducts(data))
  }, [setProducts])
  return (
    <main>
      <Head>
        <title>Create Next App</title>
        <meta name="description" content="Generated by create next app" />
        <link rel="icon" href="/favicon.ico" />
      </Head>
      <pre><code>{JSON.stringify(products,null,2)}</code></pre>
    </main>
  )
}

```

このコードでは、ページ表示後にAPIを呼び出して商品一覧情報を取得しています。

取得した結果をそのまま出力していますので、以下のように商品情報がそのまま表示されればOKです。

![](https://storage.googleapis.com/zenn-user-upload/55db090939bd-20220420.png)

## Bootstrapで見た目を整える

取得したデータを、商品一覧らしく整形しましょう。

`pages/index.js`を、以下のように編集してください。

```diff js
import Head from 'next/head'
import { useState, useEffect } from 'react'
+import { Container, Row , Col, Image, Stack, Button } from "react-bootstrap"

export default function Home() {
  const [products, setProducts] = useState([])
  useEffect(() => {
    fetch('http://localhost:3000/api/products')
      .then(response => response.json())
      .then(data => setProducts(data))
  }, [setProducts])
  return (
    <main>
      <Head>
        <title>Create Next App</title>
        <meta name="description" content="Generated by create next app" />
        <link rel="icon" href="/favicon.ico" />
      </Head>
-      <pre><code>{JSON.stringify(products,null,2)}</code></pre>
+      <Container>
+        <Stack gap={3}>
+          {products.map(product => {
+            return (
+              <Row key={product.id}>
+                <Col xs={4}>
+                  <Image
+                    src={product.images[0]}
+                    alt={product.name}
+                    style={{maxWidth: '100%'}}
+                  />
+                </Col>
+                <Col>
+                  <Stack gap={3}>
+                    <h2>{product.name}</h2>
+                    <p>{product.description}</p>
+                  </Stack>
+                  <Stack direction="horizontal">
+                    {product.prices.map(price => {
+                      return (
+                        <dl key={price.id}>
+                          <dt>価格</dt>
+                          <dd>
+                            <span>{price.unit_amount.toLocaleString()} {price.currency.toLocaleUpperCase()}</span>
+                            {price.transform_quantity ? <small>({price.transform_quantity.divide_by}アイテム毎)</small> : null}
+                          </dd>
+                          <dd>
+                            <Button>注文する</Button>
+                          </dd>
+                        </dl>
+                      )
+                    })}
+                  </Stack>
+                </Col>
+              </Row>
+            )
+          })}
+        </Stack>
+      </Container>
    </main>
  )
}

```

変更を保存すると、以下のように表示が切り替わります。

![](https://storage.googleapis.com/zenn-user-upload/67dca3afc9e6-20220420.png)


## `getStaticProps`で商品データを取得しよう

ここまでは一般的なReactアプリでのデータ取得とほとんど同じです。

Next.jsの場合、ページのビルド時やユーザーからのリクエストの際に「サーバー側で」データを取得してReact側に渡すことができます。

ここでは、ページのビルド時にデータを取得する`getStaticProps`メソッドを利用します。


`pages/index.js`を、以下のように編集しましょう。


```diff js
import Head from 'next/head'
-import { useState, useEffect } from 'react'
import { Container, Row , Col, Image, Stack, Button } from "react-bootstrap"

+export async function getStaticProps() {
+  const products = await fetch('http://localhost:3000/api/products')
+      .then(response => response.json())
+  return {
+    props: {
+      products
+    },
+  }
+}

+export default function Home({products}) {
-export default function Home() {
-  const [products, setProducts] = useState([])
-  useEffect(() => {
-    fetch('http://localhost:3000/api/products')
-      .then(response => response.json())
-      .then(data => setProducts(data))
-  }, [setProducts])
  return (
```

この変更を保存すると、ページ読み込みから商品一覧の表示までの時間差がとても短くなります。

これは、前のコードが「ページの読み込み後にAPIを呼び出している」のに対して「`getStaticProps`を使って、ビルド時点でデータの取得を済ませている」ことが要因です。


### Tips: `getStaticProps` / `getServerSideProps`でStripe SDKを実行する

今回は、REST APIへの組み込みを体験する目的もあり、REST API側でStripe SDKを実行しました。

REST API経由にすることで、`useSWR`や`fetch`を利用したクライアント側での取得にもコードを利用できるメリットがあります。

もし`getStaticProps`または`getServerSideProps`でStripe SDKを実行したい場合は、以下のように実装しましょう。

**`pages/index.js`**
```js
import Stripe from 'stripe'

async function loadStripeProduct() {
  const stripe = new Stripe(process.env.STRIPE_API_KEY, {
    apiVersion: '2020-08-27',
    maxNetworkRetries: 3,
  })
  const products = await stripe.products.list()
  if (!products.data || products.data.length < 1) {
      return res.status(200).json([])
  }
  const response = await Promise.all(products.data.map(async (product, i) => {
      const prices = await stripe.prices.list({
          product: product.id,
      })
      return {
          id: product.id,
          description: product.description,
          name: product.name,
          images: product.images,
          unit_label: product.unit_label,
          prices: prices.data.map(price => {
              return {
                  id: price.id,
                  currency: price.currency,
                  transform_quantity: price.transform_quantity,
                  unit_amount: price.unit_amount,
              }
          })
      }
  }))
  return response
}


/**
 * getServerSidePropsでもOK
 **/
export async function getStaticProps() {
  const products = await loadStripeProduct()
  return {
    props: {
      products
    }
  }
}
```

## `revalidate`を設定して、キャッシュの自動更新を行おう

`getStaticProps`を使うメリットの１つは、「ビルド時に前もってデータを取得することで、表示を早くできること」です。

ただし、Stripe側で商品データを更新した場合に、アプリ側のビルドを再実行する必要があります。

このように、Next.jsの外でデータが変わるケースに対応するためには、`revalidate`を設定します。

`pages/index.js`の`getStaticProps`に以下のコードを追加しましょう。

```diff js
export async function getStaticProps() {
  const products = await fetch('http://localhost:3000/api/products')
      .then(response => response.json())
  return {
    props: {
      products
    },
+    revalidate: 1 * 60 // 1分
  }
}
```

`getStaticProps`の戻り値に`revalidate`を秒数で設定すると、「設定時間が経過した後に、ユーザーがページを訪問すると、サーバー側でビルドを再実行」してくれます。（`next dev`で起動している環境では、常にビルドを再実します。）

これによって、サイトのデータを自動的に新しいものに変更できるようになります。この機能をNext.jsの**Incremental Static Regeneration**と呼びます。

https://nextjs.org/docs/basic-features/data-fetching/incremental-static-regeneration

変更を保存後、Stripeのダッシュボードで商品名を変更したり、商品を追加・削除すると、ページの再読み込み時に商品データが更新されていることが確認できます。

なお、この機能を利用すると、`next export`が実行できなくなりますのでご注意ください。


## おさらい

- Stripeのデータを取得するには、シークレットAPIキーまたは制限付きAPIキーを利用する
- `getStaticProps`でAPIのレスポンスを利用したページを事前にビルドできる
- `revalidate`を設定すると、一定間隔でページの再ビルドを行える

次のステップでは、商品の注文ボタンを設定します。

## 「早く終わった」という方のための、もう１ステップ

C2Cのマーケットプレイスを構築するケースやタイムセールを行いたい場合などでは、「常に最新の商品一覧を表示する」必要があります。

useSWRライブラリを組み合わせることで、最新の情報をAPIから取得し表示させることができます。

以下のドキュメントを参考に、商品一覧ページを変更してみましょう。

https://swr.vercel.app/ja/docs/with-nextjs

https://swr.vercel.app/ja/docs/revalidation