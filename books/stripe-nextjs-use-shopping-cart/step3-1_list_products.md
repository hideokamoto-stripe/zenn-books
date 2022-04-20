---
title: "Stripeに登録した商品を、Next.jsのAPIで取得しよう"
---

このステップでは、Stripeに登録した商品データをNext.jsのAPIで取得する方法を体験します。

## StripeのAPIキーを取得しよう

Stripeのデータを取得するには、APIキーを利用します。
APIキーを確認するには、ダッシュボードの[開発者]メニューを開きましょう。

![](https://storage.googleapis.com/zenn-user-upload/95097c74935d-20220419.png)

続いて、左側のメニューから[APIキー]をクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/6fce96bb18c1-20220419.png)

### Tips: Stripe 3つのAPIキー

APIキーには3つのAPIキーが存在します。

1つ目は「公開可能キー」です。これはフロントエンドアプリで、カードのtoken化などに利用します。
2つ目は「シークレットキー」です。こちらはStripeの公開しているAPI全てにアクセスができるAPIで、万が一漏洩した場合には顧客情報や注文データなどを取得されるリスクが存在します。
最後のキーは、「制限付きのキー」です。シークレットキー同様にStripeのAPIにアクセスするために利用しますが、アクセスできるリソースや操作を個別に設定することができます。

公開可能キーとシークレットキーの２つがあれば、Stripeを利用したアプリの開発が可能です。
ただし、より安全にStripeアカウントを運用したい場合は、シークレットキーの代わりに制限付きのキーを利用することをお勧めします。

### 制限付きのキーーを作成しよう

このワークショップでは、「制限付きのキー」をシークレットキーの代わりに利用します。

[制限付きのキー]セクションの[制限付きのキーを作成]をクリックしましょう。

[キーの名前]には[ec-demo]を入力しましょう。

![](https://storage.googleapis.com/zenn-user-upload/5002c678fb2c-20220419.png)

その後、[リソースタイプ]と[権限]を以下のように設定します。

|リソースタイプ|権限|
|:--|:--|
|Products|読み取り|
|Prices|読み取り|
|Checkout Sessions|書き込み|
|それ以外|なし|

ページをスクロールし、[キーを作成]をクリックすると、[制限付きのキー]セクションに[ec-demo]キーが追加されます。

![](https://storage.googleapis.com/zenn-user-upload/c00f7232fa48-20220419.png)

[...]ボタンをクリックすると、権限を後から変更したり、削除したりできます。

![](https://storage.googleapis.com/zenn-user-upload/cae5d50a2fd0-20220419.png)

最後に、[テストキーを表示]をクリックして、APIキーを取得しましょう。

![](https://storage.googleapis.com/zenn-user-upload/42f6bb803df5-20220419.png)

## Next.jsの環境変数に、APIキーを登録しよう

取得したAPIキーは、Next.jsの環境変数からアプリで利用します。

`.env.local`ファイルを作成し、以下のようにAPIキーを設定しましょう。

**.env.local**


```
STRIPE_API_KEY=rk_test_xxxxx
NEXT_PUBLIC_STRIPE_PUBLISHABLE_API_KEY=pk_test_xxxxxx
```

- `STRIPE_API_KEY`は先ほど作成した制限付きのキーを入力します。
- `NEXT_PUBLIC_STRIPE_PUBLISHABLE_API_KEY`には、公開可能キーを入力しましょう。

公開可能キーは、Reactアプリ側で利用します。そのため、`NEXT_PUBLIC_`をつけることでフロントエンドでも利用できるように設定しています。

制限付きのキーなど、第三者に見せたくない値については`NEXT_PUBLIC_`をつけないように注意しましょう。

## 商品一覧を返すREST APIを実装しよう

StripeのAPIを呼び出す準備ができましたので、さっそく呼び出してみましょう。

### Stripe SDKをインストールする

まず、`npm install stripe`で、StripeのNode SDKをインストールしましょう。

```bash
npm install stripe

...
✨  Done in 9.43s.
```

`package.json`の`dependencies`に`stripe`が追加されていればOKです。

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
+    "stripe": "^8.218.0"
```

### Stripe SDKをセットアップしよう

まずはStripe SDKをセットアップします。

`pages/api/products.js`を以下のように変更しましょう。

```diff js
+import Stripe from 'stripe'

export default function handler(req, res) {
    if (req.method.toLocaleLowerCase() !== 'get') {
        return res.status(405).end()
    }
+    const stripe = new Stripe(process.env.STRIPE_API_KEY, {
+        apiVersion: '2020-08-27', // StripeのAPIバージョンを指定
+        maxNetworkRetries: 3 // ネットワークエラーでStripe API呼び出しが失敗した時のリトライ回数を指定
+    })
    res.status(200).json([{
        name: "胡麻鯖セット",
        price: 5000,
    }, {
        name: "明太子詰め合わせ",
        price: 6000
    }])
}
  
```

ドキュメントでは、`const stripe = require('stripe')('sk_test_...')`と紹介しているケースもあります。
が、TypeScriptでは上記の書き方をする必要があるため、このワークショップではTypeScriptにあわせた実装を行います。

### 商品一覧を取得しよう

Stripe SDKでは、`new Stripe()`の戻り値に顧客や商品・料金などのリソースへアクセスするプロパティが含まれています。

商品の一覧を取得するには、`stripe.products.list()`を利用します。

https://stripe.com/docs/api/products/list

`pages/api/products.js`を以下のように変更しましょう。

```diff js
import Stripe from 'stripe'

// async関数に変更して、awaitを使えるようにする
-export default function handler(req, res) {
+export default async function handler(req, res) {
    if (req.method.toLocaleLowerCase() !== 'get') {
        return res.status(405).end()
    }
    const stripe = new Stripe(process.env.STRIPE_API_KEY, {
        apiVersion: '2020-08-27',
        maxNetworkRetries: 3,
    })
+    const products = await stripe.products.list()
+    res.status(200).json(products)
-    res.status(200).json([{
-        name: "胡麻鯖セット",
-        price: 5000,
-    }, {
-        name: "明太子詰め合わせ",
-        price: 6000
-    }])
}
  
```

変更を保存した後、`curl http://localhost:3000/api/products`を実行しましょう。
すると、APIのレスポンスがStripeダッシュボードに登録した商品データを含むJSONデータに切り替わっていることがわかります。

**`jq`を利用して整形した実行結果の例**
```bash
curl http://localhost:3000/api/products | jq .
{
  "data": [
    {
      "id": "prod_xxx",
      "object": "product",
      "active": true,
      "attributes": [],
      "created": 1650362606,
      "description": "松坂牛のステーキ２枚セットです。商品は冷凍してお届けします。",
      "images": [
        "https://files.stripe.com/links/xxxxxx"
      ],
      "livemode": false,
      "metadata": {},
      "name": "松坂牛ステーキ２枚セット",
      "package_dimensions": null,
      "shippable": null,
      "statement_descriptor": null,
      "tax_code": null,
      "type": "service",
      "unit_label": null,
      "updated": 1650362606,
      "url": null
    }
  ],
  "has_more": false,
  "url": "/v1/products"
}
```


#### Tips: List APIは最大100件まで

StripeのAPIは、`list`系のAPIで取得できる件数の最大数が100に設定されています。

そのため、101件以上の商品を取得したい場合には、再帰呼び出しが必要です。

「レスポンスの`has_more`が`true`の場合、取得できていないデータがまだある」と考えると判断しやすくなります。

### 商品に設定した料金情報も取得しよう

Stripeでは、１つの商品に複数の料金が登録できます。
そのため、商品のリストAPIの結果には料金情報が含まれていません。

今回は商品情報と料金情報の両方が必要ですので、取得した商品情報をもとに料金情報を取得しましょう。

`pages/api/products.js`を以下のように変更しましょう。

```diff js
import Stripe from 'stripe'

export default async function handler(req, res) {
    if (req.method.toLocaleLowerCase() !== 'get') {
        return res.status(405).end()
    }
    const stripe = new Stripe(process.env.STRIPE_API_KEY, {
        apiVersion: '2020-08-27',
        maxNetworkRetries: 3,
    })
    const products = await stripe.products.list()
+    if (!products.data || products.data.length < 1) {
+        return res.status(200).json([])
+    }
+    await Promise.all(products.data.map(async (product, i) => {
+        const prices = await stripe.prices.list({
+            product: product.id,
+        })
+        products.data[i].prices = prices
+    }))
    res.status(200).json(products)
}
```

はじめの３行では、商品データが０件だった場合に空の配列を返しています。

その後、商品データをループし、料金のList APIを呼び出しています。

変更を保存した後、`curl http://localhost:3000/api/products`を実行しましょう。


**`jq`を利用して整形した実行結果の例**
```bash
curl http://localhost:3000/api/products | jq .
{
  "data": [
    {
      "id": "prod_xxx",
      "object": "product",
      "active": true,
      "attributes": [],
      "created": 1650362606,
      "description": "松坂牛のステーキ２枚セットです。商品は冷凍してお届けします。",
      "images": [
        "https://files.stripe.com/links/xxxxxx"
      ],
      "livemode": false,
      "metadata": {},
      "name": "松坂牛ステーキ２枚セット",
      "package_dimensions": null,
      "shippable": null,
      "statement_descriptor": null,
      "tax_code": null,
      "type": "service",
      "unit_label": null,
      "updated": 1650362606,
      "url": null,
      "prices": {
        "object": "list",
        "data": [
          {
            "id": "price_xxxx",
            "object": "price",
            "active": true,
            "billing_scheme": "per_unit",
            "created": 1650362607,
            "currency": "jpy",
            "livemode": false,
            "lookup_key": null,
            "metadata": {},
            "nickname": null,
            "product": "prod_xxxx",
            "recurring": null,
            "tax_behavior": "unspecified",
            "tiers_mode": null,
            "transform_quantity": null,
            "type": "one_time",
            "unit_amount": 15000,
            "unit_amount_decimal": "15000"
          }
        ],
        "has_more": false,
        "url": "/v1/prices"
      }
    }
  ],
  "has_more": false,
  "url": "/v1/products"
}
```

### 必要な情報だけを返すAPIに整形しよう

これで商品・料金両方のデータが取得できました。

サイトの表示や処理には利用しないデータが多く含まれていますので、最後に必要なデータだけ返すように編集しましょう。


```diff js
import Stripe from 'stripe'

export default async function handler(req, res) {
    if (req.method.toLocaleLowerCase() !== 'get') {
        return res.status(405).end()
    }
    const stripe = new Stripe(process.env.STRIPE_API_KEY, {
        apiVersion: '2020-08-27',
        maxNetworkRetries: 3,
    })
    const products = await stripe.products.list()
    if (!products.data || products.data.length < 1) {
        return res.status(200).json([])
    }
-    await Promise.all(products.data.map(async (product, i) => {
-        const prices = await stripe.prices.list({
-            product: product.id,
-        })
-        products.data[i].prices = prices
-    }))
+    const response = await Promise.all(products.data.map(async (product, i) => {
+        const prices = await stripe.prices.list({
+            product: product.id,
+        })
+        return {
+            id: product.id,
+            description: product.description,
+            name: product.name,
+            images: product.images,
+            unit_label: product.unit_label,
+            prices: prices.data.map(price => {
+                return {
+                    id: price.id,
+                    currency: price.currency,
+                    transform_quantity: price.transform_quantity,
+                    unit_amount: price.unit_amount,
+                }
+            })
+        }
+    }))
-    res.status(200).json(products)
+    res.status(200).json(response)
}
```

これでAPIのレスポンスがシンプルになりました。
より複雑な料金体系やmetadataなどのフィールドの利用、サブスクリプションの提供などでレスポンスに含める必要のある情報が変化することに注意しましょう。


**`jq`を利用して整形した実行結果の例**
```bash
curl http://localhost:3000/api/products | jq .
[
  {
    "id": "prod_xxxxx",
    "description": "松坂牛のステーキ２枚セットです。商品は冷凍してお届けします。",
    "name": "松坂牛ステーキ２枚セット",
    "images": [
      "https://files.stripe.com/links/xxxxx"
    ],
    "unit_label": null,
    "prices": [
      {
        "id": "price_xxxx",
        "currency": "jpy",
        "transform_quantity": null,
        "unit_amount": 15000
      }
    ]
  }
]
```

**変更後の`pages/api/products.js`全文**

```js
import Stripe from 'stripe'

export default async function handler(req, res) {
    if (req.method.toLocaleLowerCase() !== 'get') {
        return res.status(405).end()
    }
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
    res.status(200).json(response)
}
  
```


## おさらい

- Stripeのデータを取得するには、シークレットAPIキーまたは制限付きAPIキーを利用する
- 第三者に見られてはダメな環境変数には、`NEXT_PUBLIC_`をつけてはいけない
- StripeのList APIは最大100件まで

次のステップでは、APIレスポンスをサイトに表示させます。

## 「早く終わった」という方のための、もう１ステップ

商品・料金情報の取得方法は、他にもいくつか存在します。

Qiitaに公開している記事を参考に、他の方法での実装にも挑戦してみましょう。

https://qiita.com/hideokamoto/items/341f220b799e51bd5752