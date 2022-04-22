---
title: "Webhookで注文内容を取得しよう"
---

ECサイトでは、注文内容を在庫システムや配送システムに共有する必要があります。

Stripeを利用する場合、注文データはWebhookで受け取ることができます。

最後のステップでは、Webhookの組み込み方法と、注文内容の取得方法を紹介します。


## テストカード番号を使って注文をテストしよう

クレジットカードやコンビニ決済などをテストするには、Stripeが用意する専用の決済情報を利用します。


クレジットカード決済の場合、以下のようにカード情報を入力すると、決済のテストができます。

- カード番号: `4242424242424242`
- 有効期限: 現在より未来の年月ならなんでもOK (`01`/`40`)
- CVC: ３桁の数字ならなんでもOK (`333`)

決済が失敗するケースや、後から不審請求をカード会社に申し立てられたケースなどもテストできます。

![](https://storage.googleapis.com/zenn-user-upload/d3685ad19019-20220420.png)

- カードの紛失による支払い拒否: `4000000000009987`
- 盗難カードによる支払い拒否: `4000000000009979`
- 商品未着による不審請求申請: `4000000000002685`

その他のテストケースについては、ドキュメントを確認しましょう。

https://stripe.com/docs/testing

## Stripe Shellを利用して、Webhookイベントを監視しよう

Webhookイベントの種類や内容を手軽に確認したい場合、ドキュメントサイトのテストモードが利用できます。

ダッシュボードにログインした状態で、[Stripe CLIのドキュメントページ](https://stripe.com/docs/stripe-cli)に移動しましょう。

[オンラインで試してみる]ボタンが表示されますので、クリックします。
![](https://storage.googleapis.com/zenn-user-upload/0344d992f943-20220420.png)


CLI操作画面がブラウザ下部に表示されます。
![](https://storage.googleapis.com/zenn-user-upload/b09650f24d2e-20220420.png)


`stripe listen`コマンドを入力しましょう。


この状態で、構築中のアプリからテスト注文を実行しましょう。
決済が完了すると、CLI画面にStripeで発生したイベントの種類とIDが表示されます。

![](https://storage.googleapis.com/zenn-user-upload/4da8adf3b6dc-20220420.png)

IDをクリックすると、ダッシュボード上でイベントの中身をみることができます。

![](https://storage.googleapis.com/zenn-user-upload/9f4097589559-20220420.png)

### Stripe CLIをインストールして、Webhookイベントを受信しよう

イベントの確認などの簡単な操作はできますが、Webhookのローカル開発にはStripe CLIのインストールが必要です。

macOSなど、`homebrew`が利用できる場合は、`brew install stripe/stripe-cli/stripe`コマンドを実行しましょう。

Windowsなどでのインストール方法は、ドキュメントをご確認ください。

https://stripe.com/docs/stripe-cli#install

その後、`stripe login`コマンドを実行しましょう。
続いて[Enter]キーを押すか、表示されているURLにアクセスします。

```bash
stripe login

Your pairing code is: assure-idol-bright-elate
This pairing code verifies your authentication with Stripe.
Press Enter to open the browser or visit https://dashboard.stripe.com/stripecli/confirm_auth?t=e9gtJxxxxxx
```

ログイン中のStripeアカウントでStripe CLIを利用するための確認画面が表示されます。

![](https://storage.googleapis.com/zenn-user-upload/3ca721a68629-20220420.png)

ターミナルに表示されている`pairing code`と画面に出ているコードが同じであることを確認して、[アクセスを許可]をクリックしましょう。

成功ページが表示されれば、接続完了です。

![](https://storage.googleapis.com/zenn-user-upload/594245480e82-20220420.png)


## Webhook APIを実装しよう

StripeのCheckoutを利用すると、決済ページで数量やアイテムを変更することも可能です。

そのため、「実際に注文された商品」を確認するためには、Webhookを利用します。

まず、Stripe CLIの`stripe listen --forward-to http://localhost:3000/api/webhooks`コマンドを実行し、Webhookイベントをローカル環境に中継しましょう。

```bash
stripe listen --forward-to http://localhost:3000/api/webhooks

⣯ Getting ready... > Ready! You are using Stripe API Version [2020-08-27]. Your webhook signing secret is whsec_xxxxx (^C to quit)
```

`whsec_xxxxx`をこの後利用しますので、コピーして環境変数に設定しましょう。

**env.local**

```diff
STRIPE_API_KEY=rk_test_xxxxx
NEXT_PUBLIC_STRIPE_PUBLISHABLE_API_KEY=pk_test_xxxx
+STRIPE_WEBHOOK_SECRET=wsec_xxx
```

Next.jsのAPIでWebhook用のAPIを追加しましょう。

まず`npm install micro`を実行します。

続いて、`pages/api/webhooks.js`を作成し、以下のコードを追加しましょう。

```js
import Stripe from 'stripe'
import {buffer} from 'micro'

const endpointSecret = process.env.STRIPE_WEBHOOK_SECRET
const stripe = new Stripe(process.env.STRIPE_API_KEY, {
  apiVersion: '2020-08-27'
})

export const config = {
    api: {
        bodyParser: false
    }
}
export default async function handler(
  request,
  response
) {
  const sig = request.headers['stripe-signature'];
  const buf = await buffer(request)

  let event;
  try {
    if (!sig) throw new Error("No signature provided")
    event = stripe.webhooks.constructEvent(buf, sig, endpointSecret);
  } catch (e) {
    const err = e instanceof Error ? e : new Error("Bad Request")
    console.log(err)
    response.status(400).send(`Webhook Error: ${err.message}`);
    return;
  }

  console.log(event)

  return response.status(200).end()
}
```

追加したコードの80％は「Stripe以外からAPIを呼び出した場合に、エラーを返す」仕組みです。
Stripeからのリクエストかを検証することで、Webhook API経由でシステムが予期せぬ挙動をしないようにガードできます。

ここで一度、`ctrl`と`c`キーを入力するなどして、実行中の`npm run dev`を停止します。
そして改めて`npm run dev`を実行して、新しい環境変数を読み込ませましょう。

再起動した状態で、テスト注文を実行すると、Stripeでイベントが発生するたびに、`npm run dev`を実行中の画面にイベントの中身が出力されます。

```bash
{
  id: 'evt_3KqbFZL8xlxrZ26g1ULN2NJa',
  object: 'event',
  api_version: '2020-08-27',
  created: 1650452349,
  data: {
    object: {
      id: 'pi_3KqbFZL8xlxrZ26g1h1NY63z',
  ...
    id: 'req_UB6i25XEVjRtsT',
    idempotency_key: 'stripe-node-retry-e35d40e2-c114-45fa-958c-33246913bf06'
  },
  type: 'payment_intent.created'
}
```

## 決済完了時のイベントから、注文内容を取得する

最後に注文内容をWebhookで取得するコードを追加しましょう。

`pages/api/webhooks.js`を以下のように変更します。

```diff js
import Stripe from 'stripe'
import {buffer} from 'micro'

const endpointSecret = process.env.STRIPE_WEBHOOK_SECRET
const stripe = new Stripe(process.env.STRIPE_API_KEY, {
  apiVersion: '2020-08-27'
})

export const config = {
    api: {
        bodyParser: false
    }
}
export default async function handler(
  request,
  response
) {
  const sig = request.headers['stripe-signature'];
  const buf = await buffer(request)

  let event;
  try {
    if (!sig) throw new Error("No signature provided")
    event = stripe.webhooks.constructEvent(buf, sig, endpointSecret);
  } catch (e) {
    const err = e instanceof Error ? e : new Error("Bad Request")
    console.log(err)
    response.status(400).send(`Webhook Error: ${err.message}`);
    return;
  }

-  console.log(event)
+  const data = event.data.object;
+  if (
+    event.type !== 'checkout.session.completed' &&
+    event.type !== 'checkout.session.async_payment_succeeded'
+  ) {
+    return response.status(200).end();
+  }
+  /**
+   * 支払いが完全に完了している場合のみ処理する
+   **/
+  if (data.payment_status === 'paid') {
+    const item = await stripe.checkout.sessions.listLineItems(data.id);
+    console.log(item)
+    /**
+     * カートの中身の情報を利用して、発送業務などのシステムを呼び出す
+     **/  
+  }

  return response.status(200).end()
}

```

変更を保存して、再度テスト注文を実行しましょう。

すると、`npm run dev`を実行中の画面に注文された商品名や数量が出力されます。


```bash

{
  object: 'list',
  data: [
    {
      id: 'li_xxxxx',
      object: 'item',
      amount_subtotal: 100,
      amount_total: 100,
      currency: 'jpy',
      description: 'コーヒードリッパー（布）',
      price: [Object],
      quantity: 1
    },
    {
      id: 'li_xxxxx',
      object: 'item',
      amount_subtotal: 1000,
      amount_total: 1000,
      currency: 'jpy',
      description: '焙煎済みコーヒー豆',
      price: [Object],
      quantity: 1
    }
  ],
...
```

実際の組み込みでは、このデータを利用して配送やCRMなどのシステムにデータを共有します。

途中の`if`文で支払い状況を確認しているのは、「コンビニ決済などでは、決済完了までに少し時間差がある」ためです。

そのため、`checkout.session.completed` または`checkout.session.async_payment_succeeded`イベントが発生した際に、支払いが完了している（＝発送可能か）を確認しています。

# おさらい

- Stripeの出来事はWebhookから受け取ることができる
- クロスセルや個数変更に備えて、Webhookで注文内容を取得しよう
- コンビニ決済など、一部の決済方法では注文と決済が別タイミングなことに注意

## 「早く終わった」という方のための、もう１ステップ

今回のワークショップでは、現在Stripeだけでは実装できない「在庫管理」についての説明を省略しています。

一方で、StripeのWebhookやuse-shopping-cartの`validateCartItems`などを利用することで、在庫システムとの連携に必要な情報やメソッドはある程度用意されています。

https://useshoppingcart.com/docs/usage/serverless/validate-cart-items

RDBMSやDynamoDB / Mongo DBなどのNoSQL DBなどを触ることができる方は、ぜひ在庫管理機能を追加することにチャレンジしてみてください。