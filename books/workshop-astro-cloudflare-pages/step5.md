---
title: "Webhookでサブスクリプションの運用を自動化しよう"
---

サブスクリプションビジネスやSaaSでは、プランの追加や変更・解約など、様々な運用イベントが発生します。

Stripeを使う場合は、Webhookを利用して「サブスクリプションや顧客の状態に変化が起きた場合」に任意の処理を実行できます。

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

## Stripe CLIをインストールして、Webhookイベントを受信しよう

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

```diff:.env
PUBLIC_STRIPE_PUBLISHABLE_API_KEY=<publishable-keyの値>
PUBLIC_STRIPE_PRICING_TABLE_ID=<pricing-table-idの値>
PUBLIC_STIPE_CUSTOMER_PORTAL_URL=<カスタマーポータルのURL>
+STRIPE_WEBHOOK_SECRET=wsec_xxx
```

続いて`POST /api/webhooks`リクエストを処理するAPIを追加します。

Astroで、`src/pages/api/webhooks.ts`ファイルを作成しましょう。

```bash
% mkdir src/pages/api
% touch src/pages/api/webhooks.ts
```

`src/pages/api/webhooks.ts`を次のように編集しましょう。

```typescript:src/pages/api/webhooks.ts
import { APIRoute } from "astro";

export const get: APIRoute = async (context) => {
    return {
        body: JSON.stringify({
            message: "Hello Astro API"
        })
    }
}
```

Astroでは、`src/pages`ディレクトリ内のファイルで、`post`や`get`などのリクエストメソッド名を持つ関数を`export`すると、対応するリクエストを処理するAPIが作れます。

そのため、http://localhost:3000/api/webhooks にアクセスすると、`Hello Astro API`を持つJSONが表示されます。

![](https://storage.googleapis.com/zenn-user-upload/5b805eae8bc3-20230425.png)

StripeのWebhookは`POST`リクエストで送信されます。

リクエストを処理するため、`post`関数も用意しましょう。

```diff typescript:src/pages/api/webhooks.ts
+export const post: APIRoute = async (context) => {
+    return {
+        body: "ok"
+    }
+}
```

## 「Stripeからのリクエスト」を検証する

Webhook用のAPIは、URLがわかれば誰でも呼び出すことができます。

「StripeがWebhookで送信してきたリクエスト」以外はHTTP 400エラーを返す処理を追加しましょう。

### Stripeダッシュボードで、シークレットAPIキーを取得しよう

StripeのSDKそしてAPIを利用するため、シークレットAPIキーを利用します。

APIキーを確認するには、ダッシュボードの[開発者]メニューを開きましょう。


![](https://storage.googleapis.com/zenn-user-upload/95097c74935d-20220419.png)

続いて、左側のメニューから[APIキー]をクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/6fce96bb18c1-20220419.png)


:::message
 Tips: Stripe 3つのAPIキー

APIキーには3つのAPIキーが存在します。

1つ目は「公開可能キー」です。これはフロントエンドアプリで、カードのtoken化などに利用します。
2つ目は「シークレットキー」です。こちらはStripeの公開しているAPI全てにアクセスができるAPIで、万が一漏洩した場合には顧客情報や注文データなどを取得されるリスクが存在します。
最後のキーは、「制限付きのキー」です。シークレットキー同様にStripeのAPIにアクセスするために利用しますが、アクセスできるリソースや操作を個別に設定することができます。

公開可能キーとシークレットキーの２つがあれば、Stripeを利用したアプリの開発が可能です。
ただし、より安全にStripeアカウントを運用したい場合は、シークレットキーの代わりに制限付きのキーを利用することをお勧めします。
:::


### 制限付きのキーを作成しよう

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

### Astroの環境変数に[制限付きのAPIキー]を設定しよう

取得したAPIキーを、`.env`に保存しましょう。

```diff:.env
PUBLIC_STRIPE_PUBLISHABLE_API_KEY=<publishable-keyの値>
PUBLIC_STRIPE_PRICING_TABLE_ID=<pricing-table-idの値>
+STRIPE_API_KEY=rk_test_xxxxx
```

:::message
**サーバー側でのみ利用する環境変数には、`PUBLIC_`をつけません。**
`PUBLIC_`から始まる環境変数で設定すると、APIキーなどの情報が漏洩する恐れがあります。
:::

### Stripe SDKで、リクエストを検証する

`src/pages/api/webhooks.ts`のAPIに、検証処理を追加します。

まずは、Stripe SDKを追加しましょう。

```bash
% npm i stripe
```

続いて、検証処理を実装します。

```diff typescript:src/pages/api/webhooks.ts
+import Stripe from 'stripe'
+const STRIPE_SECRET_API_KEY = import.meta.env.STRIPE_SECRET_API_KEY
+const STRIPE_WEBHOOK_SECRET = import.meta.env.STRIPE_WEBHOOK_SECRET

+const stripe = new Stripe(STRIPE_SECRET_API_KEY, {
+	apiVersion: '2022-11-15'
+})

export const post: APIRoute = async (context) => {
+    const { request } = context;
+    const signature = request.headers.get('stripe-signature');
+    try {
+        const body = await request.arrayBuffer();
+        const event = stripe.webhooks.constructEvent(
+            Buffer.from(body),
+          signature,
+          STRIPE_WEBHOOK_SECRET
+        );
+        return {
+            body: "ok"
+        }
+    } catch (err) {
+        const errorMessage = `⚠️  Webhook signature verification failed. ${err.+message}`
+        console.log(errorMessage);
+        return new Response(errorMessage, {
+            status: 400
+        });
+    };

-    return {
-        body: "ok"
-    }
}
```

これでStripeのWebhookから送信されたPOSTリクエスト以外では、HTTP 400エラーが返ります。

```bash
$ curl http://localhost:3000/api/webhooks -XPOST
⚠️  Webhook signature verification failed. No stripe-signature header value was provided.
```

## サブスクリプションに関する情報を受け取る準備をしよう

APIを安全に呼び出す準備ができました。

サブスクリプションで利用することの多いイベントを、いくつか受け付けてる処理を追加しましょう。

```diff typescript:src/pages/api/webhooks.ts

        const event = stripe.webhooks.constructEvent(
            Buffer.from(body),
          signature,
          STRIPE_WEBHOOK_SECRET
        );

+        switch (event.type) {
+            // 顧客の作成が成功した場合にトリガーされる
+            case 'customer.created': {
+                const customer = event.data.object
+                console.log([
+                    `顧客データが作成されました。`,
+                    `Email: ${customer.email}`
+                ].join('\n'))
+                break
+            }
+            // サブスクリプション作成が成功した時にトリガーされる
+            case 'customer.subscription.created': {
+                const subscription = event.data.object
+                console.log([
+                    `新しいサブスクリプション申し込みがありました。`,
+                    `開始日: ${subscription.start_date}`,
+                    ...subscription.items.data.map(item => `${item.price.active}: ${item.price.unit_amount} * ${item.quantity}`)
+                ].join('\n'))
+                break
+            }
+            // トライアル終了の3日前にトリガーされる
+            case 'customer.subscription.trial_will_end': {
+                const subscription = event.data.object
+                console.log([
+                    `新しいサブスクリプション申し込みがありました。`,
+                    `開始日: ${subscription.start_date}`,
+                    `トライアル終了日: ${subscription.trial_end}`,
+                    ...subscription.items.data.map(item => `${item.price.active}: ${item.price.unit_amount} * ${item.quantity}`)
+                ].join('\n'))
+                break
+            }
+            // サブスクリプション更新の数日前にトリガーされる
+            case 'invoice.upcoming': {
+                const invoice = event.data.object
+                console.log([
+                    `サブスクリプションの更新がまもなくです。`,
+                    `サブスクリプションID: ${typeof invoice.subscription === 'string' ? invoice.subscription : invoice.subscription.id}`,
+                    `請求金額: ${invoice.total}`,
+                    `請求期限: ${new Date(invoice.period_end * 1000).toISOString()}`,
+                    `請求書URL: ${invoice.hosted_invoice_url}`
+                ].join('\n'))
+                break
+            }
+            // サブスクリプションの支払いが失敗した場合
+            case 'invoice.payment_failed': {
+                const invoice = event.data.object
+                console.log([
+                    `請求書の支払いが失敗しました。`,
+                    `請求書番号: ${invoice.number}`,
+                    `請求金額: ${invoice.total}`,
+                    `請求期限: ${new Date(invoice.period_end * 1000).toISOString()}`,
+                    `請求書URL: ${invoice.hosted_invoice_url}`
+                ].join('\n'))
+            }
+            default:
+                break
+        }

        return {
            body: "ok"
        }
    } catch (err) {

```

:::details TypeScriptで型安全にWebhookイベントを処理する方法
`stripe-event-types`ライブラリを使うことで、TypeScriptの型を活かせます。

```bash
% npm i -D stripe-event-types
```

```diff typescript:src/pages/api/webhooks.ts
+/// <reference types="stripe-event-types" />
import { APIRoute } from "astro";
import Stripe from 'stripe';
...

       const event = stripe.webhooks.constructEvent(
           Buffer.from(body),
         signature,
         STRIPE_WEBHOOK_SECRET
-       );
+       ) as Stripe.DiscriminatedEvent;

```

`event.type`ごとに型が自動で切り替わりますので、複数種類のイベントを処理する際に便利です。

```typescript
        switch (event.type) {
            case 'customer.created': {
                const customer = event.data.object
                break
            }
            case 'customer.subscription.created': {
                const subscription = event.data.object
```

:::

## Stripe CLIやアプリから、動作をテストしよう

最後に、先ほどのWebhook APIが動作することを確認します。

### 請求書（サブスクリプション含む）の支払い失敗をCLIでテストする
```bash:Stripe CLIコマンド
% stripe  trigger invoice.payment_failed
```


```log:実行結果
顧客データが作成されました。
Email: null
請求書の支払いが失敗しました。
請求書番号: 06D70289-0001
請求金額: 2000
請求期限: 2023-04-26T06:38:58.000Z
請求書URL: https://invoice.stripe.com/i/acct_123445/test_Yxxxxx
```

### サブスクリプションの新規契約をCLIでテストする
```bash:Stripe CLIコマンド
% stripe  trigger customer.subscription.created
```

```log:実行結果
顧客データが作成されました。
Email: null
新しいサブスクリプション申し込みがありました。
開始日: 1682502255
true: 1500 * 1
```

## おさらい
- Stripe Webhookで、決済やサブスクリプション・請求に関する出来事を自動処理できる
- Astroでは、`src/pages/XXX.[js|ts]`でAPIも作成できる
- Stripe CLIで簡単にWebhookイベントの動作確認ができる

ここまででワークショップは終わりです。お疲れ様でした！

次のステップでは、Webhookを利用したシステムへの組み込みや自動化の例を紹介しています。

早めにワークショップが終わった方や、家に帰ってから復習・プラスアルファの学習に取り組みたい方は、ぜひ挑戦してみてください。