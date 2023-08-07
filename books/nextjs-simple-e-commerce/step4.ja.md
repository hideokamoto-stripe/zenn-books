---
title: "Step 3: Stripe APIを利用して、商品ページを動的に生成しよう"
---

無事e-commerceシステムをローンチできました。

売れ行きも順調で、新しい商品も少しずつ増えています。

## タスク: 追加した商品や販売終了した商品の反映作業を効率化しよう

ある日あなたは、運用チームから「商品を追加・削除するたびにコードスニペットを追加するのが大変」と相談されました。

また、将来的に複数の商品をまとめて注文できる「カート機能」も追加したいとのことです。

アプリケーションのコードを変更せず、カート機能の追加にも対応できるように、Stripeとの連携をアップデートしましょう。

## 0: 型情報ファイルを作成しよう

APIとReact両方で利用する、カスタムな型定義を追加します。

`app/types.ts`を作成し、次のコードを追加しましょう。

```ts:app/types.ts
import Stripe from "stripe"

export type ProductWithPrices = Stripe.Product & {
    prices: Array<Stripe.Price>
}
```

## 1: 商品情報をStripe APIから取得しよう

まずは商品情報をStripeから取得しましょう。

`app/api/prices/route.ts`を作成し、次のコードを追加します。

```ts:app/api/prices/route.ts
import { NextResponse } from "next/server";
import { ProductWithPrices } from "@/app/types";
import Stripe from "stripe";

const stripe = new Stripe(process.env.STRIPE_SECRET_API_KEY as string, {
  apiVersion: '2022-11-15'
})

export async function GET() {
    const { data: prices } = await stripe.prices.list({
        limit: 100,
        expand: ['data.product']
    })
    const products: Array<ProductWithPrices> = []
    prices.forEach(price => {
        const product = price.product as Stripe.Product
        // Stripe CLIが生成するサンプルデータを除外する
        if (product.description === '(created by Stripe CLI)') {
            return
        }
        const productIndex = products.findIndex(prod => prod.id === product.id)
        const _price = {
            ...price,
            product: product.id
        }
        if (productIndex < 0) {
            const _product: ProductWithPrices = {
                ...product,
                prices: [_price]
            }
            products.push(_product)
        } else {
            const targetProduct = products[productIndex]
            targetProduct.prices.push(_price)
            products[productIndex] = targetProduct
        }
    })
    return NextResponse.json(products)
}
```

APIをcURLで呼び出してみましょう。

```bash
 curl http://localhost:3000/api/prices 
```

次のようなレスポンスが表示されれば成功です。

```bash
[  {
    "id": "prod_OO33gMiWolXtwS",
    "object": "product",
    "active": true,
    "attributes": [],
    "created": 1691128416,
    "default_price": "price_1NbGxALQkVoOEzC2eiPDrjXm",
    "description": null,
    "images": [
      "https://files.stripe.com/links/MDB8YWNjdF8xTlpwaDhMUWtWb09FekMyfGZsX3Rlc3RfOUhFQ1ZpNHpnSFpUTXcxMUh5Z1FCRnRv00KU7V8wof"
    ],
    "livemode": false,
    "metadata": {},
    "name": "Sofa",
    "package_dimensions": null,
    "shippable": null,
    "statement_descriptor": null,
    "tax_code": null,
    "type": "service",
    "unit_label": null,
    "updated": 1691128417,
    "url": null,
    "prices": [
      {
        "id": "price_1NbGxALQkVoOEzC2eiPDrjXm",
        "object": "price",
        "active": true,
        "billing_scheme": "per_unit",
        "created": 1691128416,
        "currency": "jpy",
        "custom_unit_amount": null,
        "livemode": false,
        "lookup_key": null,
        "metadata": {},
        "nickname": null,
        "product": "prod_OO33gMiWolXtwS",
        "recurring": null,
        "tax_behavior": "unspecified",
        "tiers_mode": null,
        "transform_quantity": null,
        "type": "one_time",
        "unit_amount": 150000,
        "unit_amount_decimal": "150000"
      }
    ]
  }
]
```

:::message
**コード解説**
Stripeでは、料金と商品の両方をまとめて取得する場合、「Price APIで１度に取得する」か「Product APIとPrice APIを組み合わせて取得する」かの2つが選べます。
今回はサンプルコードとして紹介されることの少ない、「Price APIで１度に取得する」を紹介しています。

この方法では、「料金に紐づいている商品情報をまとめて取得できる」代わりに、「商品を軸にしたレスポンス生成には、配列操作が必要」なデメリットがあります。
:::

### Next.jsから、Stripe APIを呼び出す

続いて、UI（`page.tsx`）から作成したAPIを呼び出しましょう。

`app/products/page.tsx`を次のように変更します。

```diff tsx:app/products/page.tsx
import { Metadata } from 'next'
+import { ProductWithPrices } from '@/app/types'
+import { headers } from 'next/headers'
export const metadata: Metadata = {
  title: 'Products',
}
-export default function Page() {
+export default async function Page() {
+    const headersData = headers()
+    const protocol = headersData.get('x-forwarded-proto')
+    const host = headersData.get('host')
+    const apiBase = `${protocol}://${host}`
+    const products:Array<ProductWithPrices> = await fetch(`${apiBase}/api/prices`, {
+        next: {
+            revalidate: 10 * 60 // 10 minutes
+        }
+    }).then(data => data.json())
  return (
    <main className="flex min-h-screen flex-col items-center justify-between p-24">
        <div className="mb-32 grid text-center lg:mb-0 lg:grid-cols-4 lg:text-left md:gap-10">
+            <pre><code>{JSON.stringify(products,null,2)}</code></pre>
            <h1 className='text-4xl font-extrabold'>Products</h1>
            <script async
                src="https://js.stripe.com/v3/buy-button.js">
            </script>
            <stripe-buy-button
                buy-button-id="buy_btn_1xxxxx"
                publishable-key="pk_test_xxxxx"
            >    
            </stripe-buy-button>
        </div>
    </main>
  )
}
```

http://localhost:3000/products ページを再読み込みしてみましょう。

![](https://storage.googleapis.com/zenn-user-upload/98b8cc1d8d57-20230807.png)

APIから取得したデータが表示されています。

https://nextjs.org/docs/app/building-your-application/data-fetching/fetching-caching-and-revalidating


:::message
**「静的化」するか「キャッシュ」するか**

Version13以降のNext.jsでは、Stripe APIを`page.tsx`から直接呼び出すこともできます。

ただし、直接呼び出しした場合、そのデータは「アプリのビルド時`next build`のみ更新される」挙動となります。これを「静的レンダリング」とよびます。

https://nextjs.org/docs/app/building-your-application/rendering/static-and-dynamic

そのため、「商品の更新頻度が低く、手動で再ビルドも可能である」場面以外では、今回紹介しているキャッシュする手法をお勧めします。
:::

### 取得した商品情報から、商品一覧を表示しよう

取得した商品データを、Buy Buttonの代わりに配置しましょう。


`app/products/page.tsx`を次のように変更します。

```diff tsx:app/products/page.tsx
  return (
    <main className="flex min-h-screen flex-col items-center justify-between p-24">
        <div className="mb-32 grid text-center lg:mb-0 lg:grid-cols-4 lg:text-left md:gap-10">
-            <pre><code>{JSON.stringify(products,null,2)}</code></pre>
            <h1 className='text-4xl font-extrabold'>Products</h1>
-            <script async
-                src="https://js.stripe.com/v3/buy-button.js">
-            </script>
-            <stripe-buy-button
-                buy-button-id="buy_btn_1xxxxx"
-                publishable-key="pk_test_xxxxx"
-            >    
-            </stripe-buy-button>
+            {products.map(product => {
+                return (
+                    <section key={product.id} className="bg-white pb-10 rounded-lg">
+                        <Image
+                            src={product.images[0]}
+                            alt={`Product image of ${product.name}`}
+                            width={350}
+                            height={500}
+                            className='rounded-t-lg'
+                        />
+                        <div className="px-10 mt-5">
+                            <h2 className='text-xl font-bold'>{product.name}</h2>
+                            {product.prices.map(price => {
+                                if (price.recurring) return null
+                                return (
+                                    <p
+                                        key={price.id}
+                                        className="flex justify-between my-2 items-center"
+                                    >
+                                        <span>{price.unit_amount?.toLocaleString()} {price.currency}</span>
+                                        <button
+                                            className="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
+                                        >Buy now</button>
+                                    </p>
+                                )
+                            })}
+                        </div>
+                    </section>
+                )
+            })}
        </div>
    </main>
  )
```

ページを再読み込みすると、Buy Buttonから独自の商品一覧ページに表示が切り替わります。

![](https://storage.googleapis.com/zenn-user-upload/74808493c7f6-20230807.png)


## 2: Stripe Checkoutを利用して、注文ボタンを表示しよう

最後に、表示した商品を注文する処理を追加しましょう。

### 決済ページにリダイレクトするAPIを用意しよう

まずは、Stripe Checkoutのセッションを作成するAPIを用意しましょう。

`app/api/checkout/route.ts`を作成し、次のコードを追加します。

```ts:app/api/checkout/route.ts
import { NextRequest, NextResponse } from "next/server";
import Stripe from "stripe";

const stripe = new Stripe(process.env.STRIPE_SECRET_API_KEY as string, {
  apiVersion: '2022-11-15'
})

export async function POST(request: NextRequest) {
    const origin = request.headers.get('origin') || 'http://localhost:3000'
    const referer = request.headers.get('referer') || 'http://localhost:3000'

    if (request.headers.get('content-type') === 'application/x-www-form-urlencoded') {
        const body = await request.formData()
        const priceId = body.get('price_id')?.toString()
        if (!priceId) {
            return NextResponse.json({
                message: 'Cart is empty'
            }, {
                status: 400
            })
        }

        const session = await stripe.checkout.sessions.create({
            line_items: [{
                price: priceId,
                quantity: 1,
            }],
            cancel_url: referer,
            success_url: `${origin}?success=true`,
            mode: 'payment'
        })
        if (session.url) {
            return NextResponse.redirect(new URL(session.url), 303)
        } else {
            return NextResponse.json({
                message: 'Failed to create a new checkout session. Please check your Stripe Dashboard.'
            }, {
                status: 400
            })
        }
    } else if (request.headers.get('content-type') === 'application/json') {
        // If you want to add cart feature, use this section
    }
    return NextResponse.json({
        message: 'Invalid request'
    }, {
        status: 400
    })
}
```

:::message
**[応用課題] 複数商品の注文をサポートしよう**
このワークショップでは、１商品ごとの注文方法飲み紹介します。
ですが、Stripe Checkoutでは、複数種類・複数個の注文にも対応しています。

`else if (request.headers.get('content-type') === 'application/json') {}`部分を利用して、
複数個・複数種類の注文を処理する方法を考えてみましょう。
:::

### 注文ボタンから、決済ページに移動する処理を追加しよう

作成したAPIと注文ボタンを連携させましょう。

`app/products/page.tsx`を次のように変更します。

```diff tsx:app/products/page.tsx
    {product.prices.map(price => {
        if (price.recurring) return null
        return (
-            <p
+            <form
+                action={`/api/checkout`}
+                method='POST'
+                key={price.id}
                className="flex justify-between my-2 items-center"
            >
+                <input type="hidden" name="price_id" value={price.id} />
                <span>{price.unit_amount?.toLocaleString()} {price.currency}</span>
                <button
+                    type="submit"
                    className="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
                >Buy now</button>
+            </form>
-            </p>
        )
    })}
```

これで商品ごとにStripe Checkoutの注文ページに遷移する仕組みができました。

![](https://storage.googleapis.com/zenn-user-upload/0b22fb0a7d02-20230807.png)

## おさらい

- Next.jsでは、サーバー側・クライアント側の処理両方が実装できる
- Stripe APIを利用して、商品や価格情報が取得できる
- Stripe Checkoutなら、動的な決済フローやカート機能に対応できる