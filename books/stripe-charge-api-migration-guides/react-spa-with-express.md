---
title: "React SPAおよびExpress APIでの、Payment Intent切り替えガイド"
---

この章では、React SPAとExpressを利用した構成での、マイグレーション方法を紹介します。

## Step0: Charge / Token APIを利用したサンプルアプリ

コードの差分を示しながら紹介するため、まずは古いAPIを利用して実装します。

開発環境については、フロントエンド・バックエンドをまとめて管理するため、便宜上Nxを利用します。

```bash
$ yarn create nx-workspace
✔ Workspace name (e.g., org name)     · react-spa-with-express
✔ What to create in the new workspace · react-express
✔ Application name                    · Traditional commerce
✔ Default stylesheet format           · css
✔ Enable distributed caching to make your CI faster · No
```

**ライブラリのインストール**

```bash
% yarn add stripe @stripe/stripe-js @stripe/react-stripe-js
```

**環境変数の設定(.env)**

```bash
NX_API_STRIPE_PUBLISHABLE_API_KEY=pk_test_
STRIPE_SECRET_API_KEY=sk_test_
STRIPE_WEBHOOK_SECRET=whsec_
```

### Reactで決済フォームを用意する

Token / Charge APIを利用した、シンプルな決済フォームを用意しましょう。


```tsx
import { useState } from 'react'
import { CardElement, Elements, useElements, useStripe } from '@stripe/react-stripe-js'
import { loadStripe } from '@stripe/stripe-js'

// Stripe.jsの読み込み
const stripePromise = loadStripe((process.env as any).NX_API_STRIPE_PUBLISHABLE_API_KEY)

// react-stripe-jsのProviderを設定
export const App = () => {
  return (
    <Elements stripe={stripePromise}>
      <CardForm />
    </Elements>
  )
};

// 注文フォーム
const CardForm = () => {
  const [cardholderName, setCardholderName] = useState('')
  const stripe = useStripe()
  const elements = useElements()
  return (
    <form onSubmit={async (e) => {
      e.preventDefault()
      // Stripe.jsの読み込みを待つ
      if (!stripe || !elements) return
      
      // カード情報の要素を取得
      const cardElement = elements.getElement(CardElement)
      if (!cardElement) return

      // カード情報をToken化
      const { error, token } = await stripe.createToken(cardElement, {name: cardholderName})
      if (error) {
        window.alert(error.message)
        return
      }

      // Token化したカード情報を使って、決済を処理
      await fetch('/api/charge', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          token: token.id,
          amount: 500,
          currency: 'jpy',
        })
      }).then(data => data.json())
      .then((result) => {
        window.alert(`注文完了 (order_id: ${result.order_id})`)
      })
    }}>
      <fieldset>
        <legend>カード所有者</legend>
        <input type="text" value={cardholderName} onChange={e => setCardholderName(e.target.value)}/>
      </fieldset>
      <fieldset>
      <legend>
        カード番号</legend>
        <CardElement options={{hidePostalCode: true}}/>
      </fieldset>
      <button type="submit">注文する</button>
    </form>
  )
}

export default App
```

また、Express側で決済を処理するためのAPIを用意します。

```ts
import * as express from 'express'
import { Message } from '@react-spa-with-express/api-interfaces'
import { Stripe } from 'stripe'

const app = express();
app.use(express.json())

const stripe = new Stripe(process.env.STRIPE_SECRET_API_KEY, {
  apiVersion: '2020-08-27' as '2022-08-01'
})

app.post('/api/charge', async (req, res) => {
  const { token, currency, amount } = req.body
  const result = await stripe.charges.create({
    amount,
    currency,
    source: token,
    description: 'Order using Charge API'
  })

  res.status(201).json({
    order_id: result.id
  })
})

const port = process.env.port || 3333
const server = app.listen(port, () => {
  console.log('Listening at http://localhost:' + port + '/api')
})
server.on('error', console.error)
```

これでシンプルなカード決済フォームが完成しました。

![](https://storage.googleapis.com/zenn-user-upload/cdac98a7964b-20220916.png)

### Point: Token / Charge APIは、3Dセキュア2での認証ができない

この実装方法の問題点は、3Dセキュア2による認証ができないことです。

そのため、テスト環境で3Dセキュア2での認証テストに利用するカード番号`4000000000003220`を入力すると、エラーが発生します。

![](https://storage.googleapis.com/zenn-user-upload/061428c6d93f-20220916.png)

このテスト用カード番号`4000000000003220`で決済が成功するように、APIを更新していきましょう。

## Step1: Stripe SDKのアップグレード

ここからはStep0で用意したアプリをもとにコードを変更します。

古くから利用しているアプリの場合、SDKのバージョンが古いケースがあります。

そのため、まずはSDKのバージョンをアップグレードしましょう。

```bash
% yarn add stripe@latest @stripe/stripe-js@latest @stripe/react-stripe-js@latest
```

レスポンスや引数の名前が変わっているケースも一部存在します。
もし型エラーが発生している場合は、Changelogを参考に値を変更しましょう。

https://stripe.com/docs/changelog

同様にAPIバージョンも変更できます。
こちらは破壊的変更が含まれていることがあるため、今回併せて実施することはお勧めしません。
切り替え作業が完了したのち、以下のドキュメントを参考に更新作業を開始しましょう。

https://stripe.com/docs/upgrades#api-%E3%83%90%E3%83%BC%E3%82%B8%E3%83%A7%E3%83%B3

## Step2: Expressで、Payment Intentsを作成するAPIを追加する

続いて、Payment Intentsでの決済フローを実装しましょう。

Payment Intentsの場合は、事前にStripe APIを呼び出します。

そのため、Express側で新しいAPIを追加しましょう。

```ts
import { Stripe } from 'stripe';
const stripe = new Stripe(process.env.STRIPE_SECRET_API_KEY)

app.post('/api/payment_intent', async (req, res) => {
  const { currency, amount } = req.body
  try {
    const paymentIntent = await stripe.paymentIntents.create({
      amount,
      currency,
      description: "Order from PaymentIntents API"
    })
    return res.status(201).json({
      client_secret: paymentIntent.client_secret
    })
  } catch (e) {
    res.status(400).json({
      message: e.message
    })
  }
})
```

作成したAPIを実行すると、Payment Intentsのclient_secretを取得できます。

```bash
curl http://localhost:3333/api/payment_intent \
 -XPOST \
 -H "Content-Type: application/json" \
 -d '{"amount":500,"currency":"jpy"}'

{
  "client_secret": "pi_xxxxx_secret_yyyy"
}
```

## Step3: Reactで、Payment Intentsを使用した決済フローに変更する

作成したPayment Intents APIを利用して、フロントエンドの決済フローを変更しましょう。

まずはAPIの呼び出しと、client_secretをstateに保存する処理を追加します。

```diff
export const App = () => {
+  const [piClientSecret, setPiClientSecret] = useState('')
+  useEffect(() => {
+    fetch('/api/payment_intent', {
+      method: 'POST',
+      headers: {
+        'Content-Type': 'application/json',
+      },
+      body: JSON.stringify({
+        amount: 500,
+        currency: 'jpy',
+      })
+    }).then(async data => {
+      const response = await data.json()
+      if (data.ok) return response
+      throw response
+    })
+    .then((paymentIntent) => {
+      setPiClientSecret(paymentIntent.client_secret)
+    })
+    .catch(e => window.alert(e.message))
+  },[setPiClientSecret])
  return (
    <Elements stripe={stripePromise}>
-      <CardForm />
+      <CardForm piClientSecret={piClientSecret} />
    </Elements>
  )
};
```

`CardForm`側で呼び出ししても問題はありません。
今回または将来的に`<PaymentElement />`を利用する場合に備えて、`<Elements />`の外側に実装することをお勧めします。

続いて`CardForm`での決済処理を変更しましょう。

```diff
-const CardForm = () => {
+const CardForm = ({piClientSecret}) => {
  const [cardholderName, setCardholderName] = useState('')
  const stripe = useStripe()
  const elements = useElements()

  return (
    <form onSubmit={async (e) => {
      e.preventDefault()
      if (!stripe || !elements) return
      const cardElement = elements.getElement(CardElement)
-     if (!cardElement) return
+      if (!cardElement || !piClientSecret) return
+      const result = await stripe.confirmCardPayment(piClientSecret, {
+        payment_method: {
+          card: cardElement,
+          billing_details: {
+            name: cardholderName,
+          }
+        }
+      })
+      if (result.error) {
+        window.alert(result.error.message)
+        return
+      }
+      if (result.paymentIntent) {
+        window.alert(`注文完了 (order_id: ${result.paymentIntent.id})`)
+      } else {
+        window.alert(`注文完了`)
+      }
-      const { error, token } = await stripe.createToken(cardElement, {name: cardholderName})
-      if (error) {
-        window.alert(error.message)
-        return
-      }
-      await fetch('/api/charge', {
-        method: 'POST',
-        headers: {
-          'Content-Type': 'application/json',
-        },
-        body: JSON.stringify({
-          token: token.id,
-          amount: 500,
-          currency: 'jpy',
-        })
-      }).then(async data => {
-        const result = await data.json()
-        if (data.ok) return result
-        throw result
-      })
-      .then((result) => {
-        window.alert(`注文完了 (order_id: ${result.order_id})`)
-      })
-      .catch(e => window.alert(e.message))
    }}>
      <fieldset>
        <legend>カード所有者</legend>
        <input type="text" value={cardholderName} onChange={e => setCardholderName(e.target.value)}/>
      </fieldset>
      <fieldset>
      <legend>
        カード番号</legend>
        <CardElement options={{hidePostalCode: true}}/>
      </fieldset>
      <button type="submit">注文する</button>
    </form>
  )
}
```

client_secretを利用して、Stripe.jsの`confirmCardPayment`を呼び出すだけに変わりました。

submit時の処理のみ変更しましたので、決済フォームの見た目は変わりません。

### 3Dセキュア2での認証をテストする

先ほどエラーになったカード番号`4000000000003220`を利用して、3Dセキュア2での認証をテストしてみましょう。

今度はエラーメッセージではなく、認証テストのための画面が表示されます。

![](https://storage.googleapis.com/zenn-user-upload/47e110e9fd32-20220916.png)

`COMPLETE`ボタンをクリックすると、決済処理が再開されて完了メッセージが表示されます。

![](https://storage.googleapis.com/zenn-user-upload/d65f53894b8e-20220916.png)

`FAIL`をクリックした場合は、決済が失敗したメッセージが表示されます。

![](https://storage.googleapis.com/zenn-user-upload/608d40eb0381-20220916.png)


## Step4: ExpressのChargeを利用したAPIを廃止する

Charge APIを利用するAPIを使わなくなりましたので、削除しましょう。

```diff
-app.post('/api/charge', async (req, res) => {
-  const { token, currency, amount } = req.body
-  try {
-    const result = await stripe.charges.create({
-      amount,
-      currency,
-      source: token,
-      description: 'Order using Charge API'
-    })
-
-    res.status(201).json({
-      order_id: result.id
-    })
-  } catch (e) {
-    res.status(400).json({
-      message: e.message
-    })
-  }
-})
```

これでAPIの切り替え作業は完了です。

次のステップでは、より決済システムの運用保守を便利できる`PaymentElement`について紹介します。

## Step5(optional): Reactで、CardElementを、PaymentElementに変更する

現在Stripeでは、決済情報の入力に`PaymentElement`の利用を推奨しています。

これは、ユーザーの環境（地域・ブラウザなど）に応じて利用可能な決済方法を自動で複数表示できるコンポーネントです。

![](https://storage.googleapis.com/zenn-user-upload/c067946efd67-20220916.png)

決済方法によって実装を追加・変更する必要がほぼ無くなるため、Stripeが新しい決済方法のサポートを開始した場合や、新しい国・地域でサービスを開始する際の追加開発コストを減らすことができます。

https://stripe.com/docs/payments/payment-card-element-comparison

`CardElement`から`PaymentElement`への切り替えは、原則フロントエンド側の変更のみです。

### `<Elements />`にPayment Intentsのclient_secretを渡す

まず、Payment Intentsのclient_secretを`CardForm`ではなく`Elements`の`options`に渡しましょう。

`<PaymentElement />`コンポーネントは、`Elements`にclient_secretがない状態ではエラーが発生します。
そのため、`client_secret`がない状態ではローディング画面を出すように変更します。


```diff
+  if (!piClientSecret) return <p>Loading...</p>
  return (
-    <Elements stripe={stripePromise}>
+    <Elements stripe={stripePromise} options={{ clientSecret: piClientSecret }}>
-      <CardForm piClientSecret={piClientSecret} />
+      <CardForm />
    </Elements>
  )
```

### `confirmCardPayment`を`confirmPayment`に変更する

続いて`form`の`submit`処理も変更します。

```diff
-import { CardElement, Elements, useElements, useStripe } from '@stripe/react-stripe-js'
+import { Elements, PaymentElement, useElements, useStripe } from '@stripe/react-stripe-js'

-const CardForm= ({piClientSecret}) => {
+const CardForm= () => {
  const [cardholderName, setCardholderName] = useState('')
  const stripe = useStripe()
  const elements = useElements()

  return (
    <form onSubmit={async (e) => {
      e.preventDefault()
      if (!stripe || !elements) return
+      const result = await stripe.confirmPayment({
+        elements,
+        confirmParams: {
+          return_url: 'http://localhost:4200',
+          payment_method_data: {
+            billing_details: {
+              name: cardholderName,
+            }
+          }
+        },
+        redirect: 'if_required',
+      })
-      const cardElement = elements.getElement(CardElement)
-      if (!cardElement || !piClientSecret) return
-      const result = await stripe.confirmCardPayment(piClientSecret, {
-        payment_method: {
-          card: cardElement,
-          billing_details: {
-            name: cardholderName,
-          }
-        }
-      })
      if (result.error) {
        window.alert(result.error.message)
        return
      }
      if (result.paymentIntent) {
        window.alert(`注文完了 (order_id: ${result.paymentIntent.id})`)
      } else {
        window.alert(`注文完了`)
      }
    }}>
      <fieldset>
        <legend>カード所有者</legend>
        <input type="text" value={cardholderName} onChange={e => setCardholderName(e.target.value)}/>
      </fieldset>
-      <fieldset>
-      <legend>
-        カード番号</legend>
-        <CardElement options={{hidePostalCode: true}}/>
-      </fieldset>
+      <PaymentElement />
      <button type="submit">注文する</button>
    </form>
  )
}
```

`<PaymentElement />`では、コンビニや銀行振込などの「顧客が追加で支払い操作を行う必要がある決済」向けの案内表示も行います。

![](https://storage.googleapis.com/zenn-user-upload/af7106e45dd3-20220916.png)

### 銀行振込での決済を受け付ける

銀行振込での決済を受け付けるには、API側の変更が追加で必要です。

以下の記事を参考に、API側のPayment Intents作成処理を更新しましょう。

https://qiita.com/hideokamoto/items/2e5619f73cd829f48a24

## Step6: Expressで、決済完了イベントを取得するWebhook APIを追加する

`<PaymentElement />`を利用し、クレジットカード以外の決済手段をサポートすると、「その場で支払いが完了していないケース」が発生します。

コンビニ決済や銀行振込では、顧客が店舗やネットバンクを利用して入金を完了するまで、発送などの処理を待つ必要があります。

このような非同期型の決済と、クレジットカードのようにその場で決済が完了する決済両方をサポートする方法として、StripeではWebhookを利用します。

### Stripe CLIで、Webhookイベントをローカルに中継する

Stripe CLIの`stripe listen`コマンドで、StripeアカウントからのWebhookイベントをローカル環境のAPIに中継できます。

https://stripe.com/docs/stripe-cli

APIが未実装でも動作しますので、先にコマンドを実行しましょう。

```bash
$ stripe listen --forward-to http://localhost:3333/api/webhook
> Ready! You are using Stripe API Version [2020-08-27]. Your webhook signing secret is whsec_xxxxx (^C to quit)
```

`whsec_`から始まるキーが発行されますので、`.env`の`STRIPE_WEBHOOK_SECRET=`に保存しましょう。

環境変数を読み込み直すため、`npm start`や`yarn nx run-many --taret=serve --all`(Nxの場合)を再起動させる必要があります。

### Webhook APIを実装する

署名シークレットを取得できたので、APIを実装します。

コードはStripe DocsにあるコードビルダーからDLしました。

https://stripe.com/docs/webhooks/quickstart

```ts
const endpointSecret = process.env.STRIPE_WEBHOOK_SECRET
app.post('/api/webhook', express.raw({type: 'application/json'}), (request, response) => {
  let event: Stripe.Event | undefined = request.body;
  // Only verify the event if you have an endpoint secret defined.
  // Otherwise use the basic event deserialized with JSON.parse
  if (endpointSecret) {
    // Get the signature sent by Stripe
    const signature = request.headers['stripe-signature'];
    try {
      event = stripe.webhooks.constructEvent(
        request.body,
        signature,
        endpointSecret
      );
      if (!event) {
        return response.sendStatus(200)
      }
    } catch (err) {
      console.log(`⚠️  Webhook signature verification failed.`, err.message);
      return response.sendStatus(400);
    }
  }

  // Handle the event
  switch (event.type) {
    case 'payment_intent.succeeded': {
      const paymentIntent = event.data.object as Stripe.PaymentIntent;
      console.log(`PaymentIntent for ${paymentIntent.amount} was successful!`);
      // Then define and call a method to handle the successful payment intent.
      // handlePaymentIntentSucceeded(paymentIntent);
      break;
    }
    case 'payment_method.attached': {
      const paymentMethod = event.data.object as Stripe.PaymentMethod;
      console.log(`PaymentMethod: ${JSON.stringify(paymentMethod, null, 2)}`);
      // Then define and call a method to handle the successful attachment of a PaymentMethod.
      // handlePaymentMethodAttached(paymentMethod);
      break;
    }
    default:
      // Unexpected event type
      console.log(`Unhandled event type ${event.type}.`);
  }

  // Return a 200 response to acknowledge receipt of the event
  response.send();
});
```

APIの実装に成功していれば、決済完了時に`PaymentIntent for 500 was successful!`と表示されます。

このメッセージを表示している`console.log`部分に、発送システムやCRMなどのAPI呼び出し処理を追加しましょう。

`payment_intent.succeeded`のWebhookイベントは、決済手段に限らず「入金が完了した時点」で発火します。

そのためこのイベントを起点に処理を実行することで、複数の決済手段をサポートしつつ、バックオフィス系のシステムを効率化できます。

#### 注意: `express.json()`よりも前に定義しよう

Stripe SDKを利用した署名チェックでは、rawデータを利用します。

そのため、`app.use(express.json())`を実行していると、署名チェックに失敗します。

Webhook用のAPIエンドポイントまたはルートは、`express.json()`の影響を受けない形で実装する必要があります。

```diff
+ const endpointSecret = process.env.STRIPE_WEBHOOK_SECRET
+ app.post('/api/webhook', express.raw({type: 'application/json'}), (request, response) => {...})

app.use(express.json())
```

## Tips

最後に、Charge APIを利用したワークフローやカスタマイズへの対応について一部紹介します。

### 日本語での明細書表記に対応する

Charge APIでは、`alternate_statement_descriptors`を利用して日本語の明細書表記を設定できました。

```js
await stripe.charges.create({
    amount: 1000,
    currency: 'jpy',
    source: 'tok_visa',
    description: 'Order using Charge API',
    alternate_statement_descriptors: {
        kana: 'ﾒｲｻｲ',
        kanji: '明細表記'
    }
  })
```

Payment Intentsの場合は、`payment_method_options`から設定します。

```js
await stripe.paymentIntents.create({
    amount: 1000,
    currency: 'jpy',
    payment_method_types: ['card'],
    statement_descriptor_suffix: 'example descriptor',
    payment_method_options: {
      card: {
        statement_descriptor_suffix_kanji: '漢字サフィックス',
        statement_descriptor_suffix_kana: 'カナサフィックス',
      },
    },
});
```

https://qiita.com/hideokamoto/items/8eca4e1680f8aeb12ce8

### オーソリのみ実施し、決済は実行しないパターンに対応する

Charge APIでは、`capture: false`を設定してオーソリのみを実施できました。

```js
const charge = await stripe.charges.create({
    amount: 1000,
    currency: 'jpy',
    source: 'tok_visa',
    description: 'Order using Charge API',
    capture: false,
})

// 決済を実施する場合の処理
await stripe.charges.capture(charge.id)
```

Payment Intentsの場合は、`capture_method`を設定しましょう。

```js
const paymentIntent = await stripe.paymentIntents.create({
    amount: 1000,
    currency: 'jpy',
    payment_method_types: ['card'],
    capture_method: 'manual'
});

// 決済を実施する場合の処理
await stripe.paymentIntents.capture(paymentIntent.id)
```

https://stripe.com/docs/payments/place-a-hold-on-a-payment-method

### PaymentIntentsで、注文金額を変更する場合

Payment Intentsでは、フォームの入力前に決済金額や通貨を指定します。

そのため、カートページでクーポンの適用や商品数を変更できるシステムでは、Payment Intentsを更新する必要があります。

Payment Intentsの金額を変更する場合は、サーバー側でUpdate APIを実行しましょう。

```js
await stripe.paymentIntents.update('pi_xxx', {
    amount: 1500,
    metadata: {
        order_id: '6735'
    }
});
```

この処理を実行する場合、Payment Intentsを作成するAPIは**Payment IntentsのID**もレスポンスに追加する必要があります。

```diff
    return res.status(201).json({
+      id: paymentIntent.id,
      client_secret: paymentIntent.client_secret
    })
```