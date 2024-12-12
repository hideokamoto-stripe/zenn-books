---
title: "StripeのAPIキーからStripeアカウントを特定する方法"
emoji: "🔍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["hono", "stripe", "javascript", "typescript"]
published: true
---

複数のStripeアカウントで開発や運用を行っていると、プロジェクトで利用しているAPIキーがどのStripeアカウントで利用しているものかがわからなくなることがあります。

この記事では、APIキーから対象のStripeアカウントを特定する方法を紹介します。

## Checkout Sessionを利用して確認する

もっとも簡単な方法は、Checkout Sessionを利用することです。Checkout Sessionには、カード情報を保存するための`setup`モードが用意されています。`setup`モードでセッションを作成し、リダイレクトを行うAPIを実装しましょう。

```js
app.get('/checkout', async c => {
  const stripe = new Stripe(STRIPE_SECRET_API_KEY)
  const session = await stripe.checkout.sessions.create({
    mode: 'setup',
    currency: 'jpy',
    success_url: 'https://example.com/success',
    cancel_url: 'https://example.com/cancel',
  });
  return c.redirect(session.url)
})
```

このAPIにアクセスすると、Stripeが用意する決済情報保存ページにリダイレクトされます。このページでは、Stripeアカウントの名前が表示されますので、ここに表示されたアカウント名を利用してどのStripeアカウントにて発行されたAPIかを確認できます。

![](https://storage.googleapis.com/zenn-user-upload/b146017c5a8c-20241212.png)

ただしSandboxモードを利用している場合は、アカウント名ではなくサンドボックス名が表示されます。この場合はCheckout Sessionを利用してアカウントを特定することが難しくなります。

![](https://storage.googleapis.com/zenn-user-upload/ce5369696a08-20241212.png)

## Invoiceの支払いURLからアカウントIDを取得する

もう1つの方法は、Invoiceを利用します。Invoiceで発行した請求書の支払いURLは、`https://invoice.stripe.com/i/acct_xxxx/test_xxxxx`というフォーマットで作られます。このうち、`acct_xxxx`が対象のStripeアカウントIDです。このIDを取得するため、請求書の支払いURLを取得するコードを作りましょう。

請求書の支払いURLを生成するには、顧客(`customr`)・請求書(`invoice`)そして請求明細(`invoiceItem`)の3つが必要です。[Hono](https://hono.dev)を使っている場合、次のコードで取得できます。

```js
app.get('/test', async c => {
  const stripe = new Stripe(STRIPE_SECRET_API_KEY)
  const customer = await stripe.customers.create({
    email: 'test@example.com'
  })
  const invoice = await stripe.invoices.create({
    customer: customer.id,
    collection_method: 'send_invoice',
    days_until_due: 7
  })
  await stripe.invoiceItems.create({
    customer: customer.id,
    invoice: invoice.id,
    description: "test",
    currency: 'jpy',
    unit_amount: 1000,
    quantity: 1
  })
  
  const data = await stripe.invoices.finalizeInvoice(invoice.id)
  return c.text(data.hosted_invoice_url)
})
```

追加したAPIをcurlで呼び出してみましょう。先ほど紹介したフォーマットのURLがレスポンスで取得できます。


```bash
$ curl http://localhost:8787/test
"https://invoice.stripe.com/i/acct_xxxx/test_xxxxx"
```
あとは取得したアカウントIDを利用して、アカウント情報を取得しましょう。

```js
const stripe = new Stripe(STRIPE_SECRET_API_KEY)
const account = await stripe.accounts.retrieve('acct_xxx')
console.log(account)
```

これでアカウントのプロフィール名やメールアドレスが特定できます。

```json
{
  "id": "acct_xxx",
  "object": "account",
  "business_profile": {
    "annual_revenue": null,
    "estimated_worker_count": null,
    "mcc": null,
    "name": "Sandbox",
    ...
  },
  "email": "test@example.com",
```


## まとめ

少し特殊な使い方ではありますが、プロジェクトに紐づけられたAPIキーがどのStripeアカウントで発行されたかを特定するために、Stripe API / SDKを利用できます。

アカウント名やID・メールアドレスの情報を取得することができれば、あとは[Stripeダッシュボードの設定ページ](https://dashboard.stripe.com/test/settings/account)や[サンドボックス管理画面](https://dashboard.stripe.com/sandboxes)にて該当のアカウントを特定しましょう。
