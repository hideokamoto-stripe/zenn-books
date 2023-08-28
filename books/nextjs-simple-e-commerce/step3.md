---
title: "Step 2: Next.js REST API の開発と注文処理の統合"
---

Step1で、商品登録と表示、そして注文ページへの動線まで用意できました。

e-commerceサイトでは、顧客が商品を注文した後に、商品の準備や発送を行う必要があります。

社内の発送システムに注文情報を送信するため、Stripeで受け付けた注文を外部システムに連携する「Webhook API」を開発しましょう。

:::message
**「社内システムとの連携」についての補足**
ワークショップでは、「Stripeから送信された注文情報を受け取るAPI」までを開発します。
「注文情報を受け取る仕組みまでを用意し、実際の連携作業は別チームが担当する」ような開発フロー・・・ということで進めましょう。
:::

## Next.js(v13 App Router)で、REST APIを追加する

まずは、APIを作成しましょう。

`app/api/webhook/route.ts`ファイルを作成し、次のコードを追加します。

```ts:app/api/webhook/route.ts
import { NextResponse } from "next/server";

export async function POST() {
  return NextResponse.json({
      message: "Hello! Stripe Webhook."
  })
 }
```

Next.jsのApp Routerでは、「`ディレクトリ名`がAPIのパス」になり、「`route.ts`が配置されていれば、ファイル内のメソッドに対応する関数（`POST`, `GET`など）が実行される」動きをします。

https://nextjs.org/docs/app/building-your-application/routing

今回は、`/api/webhook`に対する`POST`リクエストを処理するので、`app/api/webhook/route.ts`ファイルに`POST`関数を配置しました。

次のようなcURLコマンドを実行して、`"Hello! Stripe Webhook."`が表示されれば成功です。

```bash
 curl -XPOST http://localhost:3000/api/webhook
```

:::message
**Tips: `route.ts`と`page.tsx`を同一ディレクトリに配置しない**

- `GET /dummy`は、UIを表示する
- `POST /dummy`は、REST APIを呼び出す

このような設計にするには、`app/dummy`ディレクトリに`route.ts`と`page.tsx`両方を配置することになります。
しかし2023/08時点では、この2ファイルを同時に配置することはできません。

https://nextjs.org/docs/app/building-your-application/routing/route-handlers#route-resolution

衝突リスクを減らすためにも、Next.js v12以前同様APIは`api`ディレクトリ配下にまとめることをお勧めします。
:::

### リクエスト内容を確認する方法

Stripeなどから送信されたデータを確認するには、`POST`関数の引数をみます。

`app/api/webhook/route.ts`ファイルを、次のように変更しましょう。

```diff ts:app/api/webhook/route.ts
-export async function POST() {
+export async function POST(request: Request) {
+  const body = await request.json()
+  console.log(JSON.stringify(body, null, 2))
  return NextResponse.json({
-      message: "Hello! Stripe Webhook."
+      message: `Hello ${body.name ?? "there"}!`
  })
 }
```

JSONで送信されたデータは、`request.json()`から取得できます。

次のcURLコマンドで、APIを呼び出してみましょう。

```bash
 curl -XPOST http://localhost:3000/api/webhook -d '{"name": "John"}'
 ```

送信した名前がレスポンスに含まれていることが確認できます。

 ```bash
{
  "message": "Hello John!"
}
```

## Stripe CLIで、StripeのWebhookイベントをNext.jsに中継する

APIの準備ができました。

次はこのAPIに、Stripe上で発生したイベントを送信する設定を行います。

StripeのWebhookを利用することで、決済の完了や失敗・サブスクリプションの状態変化などの様々なイベントを、外部のシステムに送信できます。

開発中は、Stripe CLIの`--forward-to`オプションを利用して、ローカルのAPIにStripe Webhookのイベントを中継できます。

次のコマンドを実行しましょう。

```bash
stripe listen --forward-to http://localhost:3000/api/webhook
```

`Ready`と表示されれば、成功です。

```
> Ready! You are using Stripe API Version [2022-11-15]. Your webhook signing secret is whsec_xxxxxx (^C to quit)
```

### Stripe CLIを利用して、Next.jsへの中継が成功しているか確認しよう

Next.jsで作成したAPIにイベントが届いているかを確認しましょう。

Stripe CLIからWebhookイベントを送信します。

```bash
stripe trigger checkout.session.completed
```

次のようにStripeのAPI呼び出しのログがCLI実行画面に表示されます。

```bash
Setting up fixture for: product
Running fixture for: product
Setting up fixture for: price
Running fixture for: price
Setting up fixture for: checkout_session
Running fixture for: checkout_session
Setting up fixture for: payment_page
Running fixture for: payment_page
Setting up fixture for: payment_method
Running fixture for: payment_method
Setting up fixture for: payment_page_confirm
Running fixture for: payment_page_confirm
Trigger succeeded! Check dashboard for event details.
```

また、`stripe listen`を実行しているターミナル画面では、Webhookの送信結果が表示されます。

```bash
2023-08-07 12:04:06   --> product.created [evt_1NcJjmLQkVoOEzC2Fbwk9riN]
2023-08-07 12:04:06  <--  [200] POST http://localhost:3000/api/webhook [evt_1NcJjmLQkVoOEzC2Fbwk9riN]
```

そして`next dev`や`npm run dev`を実行しているターミナル画面に、次のようなイベントログが表示されれば成功です。

```log
{
  "id": "evt_3NcJjpLQkVoOEzC20xKExGCI",
  "object": "event",
  "api_version": "2022-11-15",
  "created": 1691377449,
  "data": {
    "object": {
      "id": "pi_3NcJjpLQkVoOEzC20n0doDzG",
      "object": "payment_intent",
      "amount": 3000,
      "amount_capturable": 0,
      "amount_details": {
        "tip": {}
      },
...
```

これでStripeアカウント内で発生した様々なイベントと、開発中のシステムとの連携の準備ができました。

## APIリクエストが、「Stripeから送信されたものか」を検証する

StripeのようなSaaSから送信されるWebhookとシステムを連携する場合、「そのサービス以外からのAPIリクエストを拒否」しなければなりません。

誰でもAPIを呼び出せる状態では、「偽の注文イベントを送信する」などの不正利用を受ける恐れがあります。

**偽の注文データを送信する例**
```bash
curl -XPOST http://localhost:3000/api/webhook -d '{ "id": "evt_3NcJjpLQkVoOEzC20xKExGCI", "object": "event", "api_version": "2022-11-15", "created": 1691377449, "data": { "object": { "id": "pi_3NcJjpLQkVoOEzC20n0doDzG", "object": "payment_intent", "amount": 3000, "amount_capturable": 0, "amount_details": { "tip": {} } } } }' -H 'Content-Type: application/json'
```

Stripeでは、Webhook用の署名シークレットとシークレットAPIキーの2つを利用して、リクエストを検証します。

### 署名シークレットを取得する

先ほどの`stripe listen`コマンド実行結果を確認しましょう。

```
> Ready! You are using Stripe API Version [2022-11-15]. Your webhook signing secret is whsec_xxxxxx (^C to quit)
```

`Your webhook signing secret is whsec_xxxxxx`と書かれているように、`whsec_xxx`がローカル開発時の署名シークレットです。

`.env.local`ファイルを作成し、この値をコピーして保存しましょう。

```env:.env.local
STRIPE_WEBHOOK_SECRET=wsec_xxx
```

### StripeのシークレットAPIキーを、環境変数に追加する

署名検証処理では、StripeのシークレットAPIキーも必要です。

APIキーを確認するには、ダッシュボードの[開発者]メニューを開きましょう。

![](https://storage.googleapis.com/zenn-user-upload/95097c74935d-20220419.png)

続いて、中段のメニューから[API Keys]をクリックしましょう。
![](https://storage.googleapis.com/zenn-user-upload/25080b93fe5b-20230807.png)

:::message
**Tips: Stripe 3つのAPIキー**

APIキーには3つのAPIキーが存在します。

1つ目は「公開可能キー」です。これはフロントエンドアプリで、カードのtoken化などに利用します。
2つ目は「シークレットキー」です。こちらはStripeの公開しているAPI全てにアクセスができるAPIで、万が一漏洩した場合には顧客情報や注文データなどを取得されるリスクが存在します。
最後のキーは、「制限付きのキー」です。シークレットキー同様にStripeのAPIにアクセスするために利用しますが、アクセスできるリソースや操作を個別に設定することができます。

公開可能キーとシークレットキーの２つがあれば、Stripeを利用したアプリの開発が可能です。
ただし、より安全にStripeアカウントを運用したい場合は、シークレットキーの代わりに制限付きのキーを利用することをお勧めします。
:::


[標準キー]セクションの[シークレットキー]が、サーバー側で利用するAPIキーです。


![](https://storage.googleapis.com/zenn-user-upload/b166ee3d52dd-20230807.png)

[テストキーを表示]ボタンをクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/0b2508bce444-20230807.png)

シークレットAPIキーが表示されました。


![](https://storage.googleapis.com/zenn-user-upload/aa96f0a93687-20230807.png)

表示されているAPIキーをクリックするだけで、シークレットAPIキーをコピーできます。

`.env.local`ファイルに、取得したAPIキーを追加しましょう。


```diff:.env.local
STRIPE_WEBHOOK_SECRET=wsec_xxx
+STRIPE_SECRET_API_KEY=rk_test_xxxxx
```

### Install Stripe SDK

最後に、検証処理に利用するSDKをインストールしましょう。

```bash
npm i stripe
```

これで準備ができました。

### Next.jsのAPIに、署名検証処理を追加する

取得した署名シークレットを利用して、Stripe WebhookからのAPI呼び出しかどうかを検証する処理を追加しましょう。

`app/api/webhook/route.ts`を次のように書き換えてください。


```ts:app/api/webhook/route.ts
import { NextResponse } from "next/server";
import Stripe from "stripe";

const stripe = new Stripe(process.env.STRIPE_SECRET_API_KEY as string, {
  apiVersion: '2022-11-15'
})

export async function POST(request: Request) {
  const signature = request.headers.get("stripe-signature");
  if (!signature) {
    return NextResponse.json({
      message: 'Bad request'
    }, {
      status: 400
    }) 
  }
  try {
    const body = await request.arrayBuffer();
    const event = stripe.webhooks.constructEvent(
      Buffer.from(body),
      signature,
      process.env.STRIPE_WEBHOOK_SECRET as string
    );
    console.log({
      type: event.type,
      id: event.id,
    })
    return NextResponse.json({
      message: `Hello Stripe webhook!`
    });
  } catch (err) {
    const errorMessage = `⚠️  Webhook signature verification failed. ${(err as Error).message}`
    console.log(errorMessage);
    return new Response(errorMessage, {
      status: 400
    })
  }
}
```
:::message
**コード解説**
まずヘッダーから`stripe-signature`を取得します。これはStripeがAPIを呼び出す際に付与したものです。

この値と、APIリクエストのBodyを利用して、「Stripeからのリクエストか否か」を検証します。

なお、「Next.jsの、`request.json()`や`request.text()`で取得できるBody」は、アプリ内での取り扱いを簡単にするための処理がされています。

「Stripe SDKが利用するBody」では、処理される前のBodyが必要ですので、`Buffer`などを利用して元に戻す処理を追加しています。
:::

変更を保存後、Stripe CLIからWebhookイベントを送信し、Next.jsのAPIにエラーが発生しなければ成功です。

**1: イベントを送信するコマンド**
```bash
 stripe trigger payment_intent.succeeded
```

**2: `stripe listen`コマンド実行中のターミナルに表示されるログの例**

```bash
2023-08-07 14:11:59  <--  [200] POST http://localhost:3000/api/webhook [evt_3NcLjVLQkVoOEzC20HbqByIJ]
```

**3: `next dev` / `npm run dev`コマンド実行中のターミナルに表示されるログの例**

```bash
{ type: 'charge.succeeded', id: 'evt_3NcLjVLQkVoOEzC20HbqByIJ' }
```

APIを直接呼び出してみましょう。

```bash
curl -XPOST http://localhost:3000/api/webhook -d '{"name": "John"}'
```

エラーになることがわかります。

```bash
{
  "message": "Bad request"
}
```

これでStripeと連携するAPIを安全に運用できる準備ができました。


## 注文内容を表示する処理を実装しよう

社内の発送システムとの連携を行うチームに引き継ぎするため、「注文された商品や顧客情報」を取得する処理を実装しましょう。

### サポートするWebhookイベント

Stripeのリダイレクト型決済フォーム（Checkout / Payment Links + Buy button & Pricing Table）を利用している場合、注文が完了したことを知らせるイベントは次の3つです。

- `checkout.session.completed`: Checkoutの注文フローが完了した状態を知らせるイベント
- `checkout.session.async_payment_succeeded`: 銀行振込やコンビニ決済など「注文と決済が同時でないケース」にて、顧客の決済が完了した状態を知らせるイベント
- `checkout.session.async_payment_failed`: 「期日までに、指定のコンビニエンスストアまたは銀行口座に入金がなかった」場合など、支払いが完了しなかった状態を知らせるイベント

3つのイベント以外では、処理を行わないようにコードを変更しましょう。

`app/api/webhook/route.ts`を次のように変更します。

```diff ts:app/api/webhook/route.ts
    const event = stripe.webhooks.constructEvent(
      Buffer.from(body),
      signature,
      process.env.STRIPE_WEBHOOK_SECRET as string
    );
-    console.log({
-      type: event.type,
-      id: event.id,
-    })
+    if ([
+      'checkout.session.completed',
+      'checkout.session.async_payment_succeeded',
+    ].includes(event.type)) {
+      // 決済が成功したケース
+    } else if (event.type === 'checkout.session.async_payment_failed') {
+      // 決済が失敗したケース
+    }
    return NextResponse.json({
      message: `Hello Stripe webhook!`
    });
```

### 「支払いが完了した注文」のみを処理するように実装しよう

利用できる決済手段を増やすことは、e-commerceサイトを利用する顧客層の拡大にもつながります。

クレジットカード以外の決済もサポートできるようにするため、「決済まで完了した注文のみ、発送処理を開始する」仕組みを用意しましょう。

銀行振込などの「顧客による追加のアクションが必要な決済」では、`checkout.session.completed`イベントが発生した時点で、決済が完了していません。

ただし、クレジットカード決済など、その場で決済まで完了しているイベントでは、`checkout.session.async_payment_succeeded`イベントは発生しません。

そのため、両方のイベントをサポートする処理が必要です。

`app/api/webhook/route.ts`を次のように変更しましょう。

```diff ts:app/api/webhook/route.ts
    const event = stripe.webhooks.constructEvent(
      Buffer.from(body),
      signature,
      process.env.STRIPE_WEBHOOK_SECRET as string
    );
    if ([
      'checkout.session.completed',
      'checkout.session.async_payment_succeeded',
    ].includes(event.type)) {
      // 決済が成功したケース
+      const data = event.data.object as Stripe.Checkout.Session
+      if (data.payment_status === 'paid') {

+      }
    } else if (event.type === 'checkout.session.async_payment_failed') {
      // 決済が失敗したケース
    }
    return NextResponse.json({
      message: `Hello Stripe webhook!`
    });
```

これで、「支払いステータスが`paid`ではない注文は処理しない」動きをサポートできました。

`checkout.session.completed`と`checkout.session.async_payment_succeeded`両方をサポートすることで、複数の決済手段にもサポートしやすくなっています。

### Webhookイベントデータから、顧客情報を取得する

発送システムに連携する条件設定が完了しましたので、送信する情報を取得します。

顧客情報については、イベントデータから取得できます。


`app/api/webhook/route.ts`を次のように変更しましょう。

```diff ts:app/api/webhook/route.ts
    if ([
      'checkout.session.completed',
      'checkout.session.async_payment_succeeded',
    ].includes(event.type)) {
      // 決済が成功したケース
      const data = event.data.object as Stripe.Checkout.Session
      if (data.payment_status === 'paid') {
+        const customerDetails = data.customer_details
+        console.log({
+          name: customerDetails?.name,
+          address: customerDetails?.address,
+          email: customerDetails?.email,
+          phone: customerDetails?.phone,
+          amount_total: data.amount_total,
+          currency: data.currency
+        })
      }
    } else if (event.type === 'checkout.session.async_payment_failed') {
      // 決済が失敗したケース
    }
```

商品注文イベントを、Stripe CLIで送信してみましょう。

```bash
stripe trigger checkout.session.completed
```

Next.jsアプリのログに、注文金額や住所が表示されます。

```bash
{
  name: 'Jenny Rosen',
  address: {
    city: 'South San Francisco',
    country: 'US',
    line1: '354 Oyster Point Blvd',
    line2: null,
    postal_code: '94080',
    state: 'CA'
  },
  email: 'stripe@example.com',
  phone: null,
  amount_total: 3000,
  currency: 'usd'
}
```

### Stripe APIでカートの中身を取得する

最後にカートの中身についても取得しましょう。

こちらは、Webhookイベントの「Checkout Session ID」を利用して、Stripe APIを呼び出します。

`app/api/webhook/route.ts`を次のように変更しましょう。

```diff ts:app/api/webhook/route.ts
    if ([
      'checkout.session.completed',
      'checkout.session.async_payment_succeeded',
    ].includes(event.type)) {
      // 決済が成功したケース
      const data = event.data.object as Stripe.Checkout.Session

      if (data.payment_status === 'paid') {
        const customerDetails = data.customer_details
        console.log({
          name: customerDetails?.name,
          address: customerDetails?.address,
          email: customerDetails?.email,
          phone: customerDetails?.phone,
          amount_total: data.amount_total,
          currency: data.currency
        })
+        const { data: cartItems } = await stripe.checkout.sessions.listLineItems(data.id, {
+          expand: ['data.price.product']
+        })
+        cartItems.forEach(item => {
+          const product = (item.price?.product as Stripe.Product)
+          console.log({
+            product: product.name,
+            unit_amount: item.price?.unit_amount,
+            currency: item.price?.currency,
+            quantity: item.quantity,
+            amount_total: item.amount_total
+          })
+        })  
      }
    } else if (event.type === 'checkout.session.async_payment_failed') {
      // 決済が失敗したケース
    }
```

商品注文イベントを、Stripe CLIで送信してみましょう。

```bash
stripe trigger checkout.session.completed
```

Next.jsアプリのログに、カートの商品情報が表示されました。

```bash
{
  product: 'myproduct',
  unit_amount: 1500,
  currency: 'eur',
  quantity: 2,
  amount_total: 3000
}
```

あとはこれらのデータを利用して、引き継ぎしたチームが開発を続けるだけです。

## おさらい

- Next.js(v13)のApp Routerでは、`app/[パス名]/route.ts`でAPIを追加できる
- どのメソッドをサポートするかは、`export function [メソッド名]`で定義できる
- Stripe CLIを使えば、StripeのWebhookイベントをローカル環境に転送できる
