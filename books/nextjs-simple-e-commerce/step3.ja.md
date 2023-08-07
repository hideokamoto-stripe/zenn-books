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

## Stripe CLIで、StripeのWebhookイベントをNext.jsに中継する

APIの準備ができました。

次はこのAPIに、Stripe上で発生したイベントを送信する設定を行います。

StripeのWebhookを利用することで、決済の完了や失敗・サブスクリプションの状態変化などの様々なイベントを、外部のシステムに送信できます。

開発中は、Stripe CLIの`--forward-to`オプションを利用して、ローカルのAPIにStripe Webhookのイベントを中継できます。

次のコマンドを実行しましょう。

```bash
stripe listen --forward-to http://locahost:3000/api/webhook
```

`Ready`と表示されれば、成功です。

```
> Ready! You are using Stripe API Version [2022-11-15]. Your webhook signing secret is whsec_xxxxxx (^C to quit)
```

### Stripe CLIを利用して、Next.jsへの中継が成功しているか確認しよう

Next.jsで作成したAPIにイベントが届いているかを確認しましょう。

Stripe CLIからWebhookイベントを送信します。

```bash

```

`next dev`や`npm run dev`を実行しているターミナル画面に、次のようなイベントログが表示されれば成功です。

```log


```

## APIリクエストが、「Stripeから送信されたものか」を検証する

StripeのようなSaaSから送信されるWebhookとシステムを連携する場合、「そのサービス以外からのAPIリクエストを拒否」しなければなりません。

誰でもAPIを呼び出せる状態では、「偽の注文イベントを送信する」などの不正利用を受ける恐れがあります。

```bash
// 偽の注文データを送信する例
@TODO
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
+STRIPE_API_KEY=rk_test_xxxxx
```

これで準備ができました。

### Next.jsのAPIに、署名検証処理を追加する

取得した署名シークレットを利用して、Stripe WebhookからのAPI呼び出しかどうかを検証する処理を追加しましょう。

```diff ts:t.ts

```

Stripe CLIからWebhookイベントを送信し、Next.jsのAPIにエラーが発生しなければ成功です。

```bash

```

APIを直接呼び出そうとすると、エラーになることがわかります。

```bash


```


これでStripeと連携するAPIを安全に運用できる準備ができました。


## 注文内容を表示する処理を実装しよう

## おさらい

- Next.js(v13)のApp Routerでは、`app/[パス名]/route.ts`でAPIを追加できる
- どのメソッドをサポートするかは、`export function [メソッド名]`で定義できる
- Stripe CLIを使えば、StripeのWebhookイベントをローカル環境に転送できる
