---
title: "[Advanced] Webhookを使ったシステム連携について考えてみよう"
---

もし想定していた時間よりも早く作業が終わった場合、ぜひ「Webhookを使った、運用業務やシステム連携の自動化」についてアイディア出しや組み込みサンプルの作成に挑戦してみてください。

参考までに、Cloudflareや各種SaaSも組み合わせた、サブスクリプション・SaaSシステムで見かけることの多いケースをいくつか紹介します。

## システムのログインユーザー情報と、StripeのCustomer情報を紐づける

複数のプランを提供する場合や、サブスクリプション契約を複数行う設計では、ユーザー情報とStripeのCustomer情報を紐づける必要があります。

### サブスクリプション申込後に、ユーザーアカウントを作成する場合

サブスクリプションの申し込みが完了してから、ユーザーアカウントを発行する方法では、Webhookを利用します。。

このケースでは、`customer.created`や`customer.subscription.created`、`checkout.session.completed`のWebhookイベントから、顧客やサブスクリプションの作成時・Checkoutのセッション完了時にユーザーアカウントを発行しましょう。

```diff typescript:Customer作成時に、ユーザーアカウントを作る例
        switch (event.type) {
            // 顧客の作成が成功した場合にトリガーされる
            case 'customer.created': {
                const customer = event.data.object
+                const payload = {
+                    name: customer.name,
+                    email: customer.email,
+                    phone_number: customer.phone,
+                }
+                await AnyUserService.createNewUser(payload)
                console.log([
                    `顧客データが作成されました。`,
                    `Email: ${customer.email}`
                ].join('\n'))
                break
            }

```

この方法を採用すると、Webhook APIを用意せずにZapierやAWS Step Functionsなどでワークフローを構築できるメリットもあります。

### ユーザー登録後にサブスクリプションを契約する場合

先にIDP側にアカウントがある場合、そのユーザー情報を料金表のタグに渡して連携します。

`client-reference-id`にユーザー名やIDを設定することで、WebhookやCheckoutのセッション履歴から「どのユーザーの申し込みか」を判別できます。

```diff html:参照IDを利用する
    <stripe-pricing-table
      pricing-table-id="'{{PRICING_TABLE_ID}}'"
      publishable-key="pk_test_xxxxxxv"
+      client-reference-id="{{CUSTOMER_USERNAME}}"
    >
    </stripe-pricing-table>
```

また、アカウントのメールアドレスが取得できる場合は、`customer-email`でメールアドレスも設定しましょう。

これによって、メールアドレスが事前入力された状態でサブスクリプション申込フローを開始できます。

```diff html:決済ページのメールアドレスを入力済みにする
    <stripe-pricing-table
      pricing-table-id="'{{PRICING_TABLE_ID}}'"
      publishable-key="pk_test_xxxxxxv"
+      customer-email="{{CUSTOMER_EMAIL}}"
    >
    </stripe-pricing-table>
```


#### [Appendix] IDPとのデータ紐付けを行わずに連携する方法
LINE Loginなど、一部のIDPサービスではメタデータなどを利用したCustomer IDの保存ができません。
その場合、メールアドレスやIDP側のユーザーIDを利用する方法も検討しましょう。

```typescript:EmailでStripeの顧客情報を取得する
const { data: [ customer ] } = await stripe.customers.list({
    email: 'test@example.com'
})
```

ユーザーIDなどを利用したい場合は、metadataに保存した上でSearch APIを利用します。

```typescript:MetadataでStripeの顧客情報を取得する
const { data: [ customer ] } = await stripe.customers.search({
    query: `metadata['userId']:'${userId}'`
})
```

ただしこちらの方法は、**1 秒あたり最大 20 件の読み取り操作のレート制限**がありますので、ユーザー数が増加する前に[Amazon DynamoDB](https://aws.amazon.com/jp/dynamodb/)や[Cloud firestore](https://firebase.google.com/docs/firestore?hl=ja), [momento](https://jp.gomomento.com/)などを利用した中間データベースを作成することをお勧めします。


## 顧客や社内に、イベントと連動したメールを送信する

SaaSにおいて、メールを利用したコミュニケーションは重要な施作の1つです。

トライアルユーザーや無料プランユーザーに対して、プランのアップグレードを提案するメールを送信する。

月額プランで1年以上契約しているユーザーには、年額プランが安くなることを提案。

未払いやカードの有効期限・年額プランの更新時期には、ユーザーと社内のサポートチームに情報を共有する。

などなど・・・

Cloudflare PagesやWorkersを利用している場合、MailChannelsを利用してメールを送信できます。

https://qiita.com/khayama/items/87640de36b72d2ceaade

```ts
export const post: APIRoute = async (context) => {
    return withStripeWebhookVerification(context, async (event) => {
        const body = event
        switch (event.type) {
            // 顧客の作成が成功した場合にトリガーされる
            case 'customer.created': {
                const customer = event.data.object
                await fetch('https://api.mailchannels.net/tx/v1/send', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify({
                        personalizations: [{
                            to: [{
                                email: customer.email,
                                name: customer.name,
                            }]
                        }],
                        from: {
                            email: import.meta.env.FROM_EMAIL,
                            name: 'XXXサービス'
                        },
                        subject: 'ようこそXXXサービスへ！',
                        content: [
                            {
                                type: 'text/plain',
                                value: `オンボーディングのためのコンテンツ`,
                            },
                        ],
                    })
                })
                break
            }
```

SendGridやAmazon SESなどを利用することや、HubSpot / SalesforceなどのSFA・CRMを経由することも可能です。

### 注文内容・数量を取得する

料金表やPayment Links・Checkoutなどのリンク型・リダイレクト型決済フォームを利用した場合、「クロスセルによる商品の追加提案」や「決済ページでの注文数量の変更」などが行えます。

https://qiita.com/hideokamoto/items/bc990f9211ede447b74e

この場合、Webhookを利用して「実際に顧客が注文した商品と数量」を把握する必要があります。

料金表・Payment Links・Checkoutいずれの場合でも、`checkout.session.completed`と`checkout.session.async_payment_succeeded`を利用します。

```diff typescript:注文商品データを取得するサンプルコード
        const body = event
        switch (event.type) {
+            case 'checkout.session.completed': {
+                const session = event.data.object
+                const lineItems = await stripe.checkout.sessions.listLineItems(session.id)
+                lineItems.data.forEach(item => {
+                    const productId = typeof item.price.product === 'string' ? item.price.product : item.price.product.id
+                    console.log([
+                        `商品ID: ${productId}`,
+                        `注文数量: ${item.quantity}`,
+                        `金額: ${item.amount_total}`
+                    ].join('\n'))
+                })
                break
            }
```

```log:カートの中身を出力している例
顧客データが作成されました。
Email: test@example.com
新しいサブスクリプション申し込みがありました。
開始日: 1682574904
true: 1400 * 1
商品ID: prod_LFzTo8Pi8xirKs
注文数量: 1
金額: 1400
商品ID: prod_LFzdDEOEwRtuRw
注文数量: 1
金額: 2000
```

3DSによる追加の認証やコンビニ決済など、「決済が完了するまで少し時間がかかるケース」が存在します。

決済完了が非同期の場合は、`checkout.session.async_payment_succeeded`イベントで改めて決済完了後にイベントが送信されます。

`checkout.session.completed`にて、`payment_status`が`paid`かどうかを判定しましょう。

```diff typescript:非同期型決済にもサポートする例
-            case 'checkout.session.completed': {
+            case 'checkout.session.completed':
+            case 'checkout.session.async_payment_succeeded': {
                const session = event.data.object
+                if (session.payment_status !== 'paid') {
+                    break
+                }
```


## おさらい

- リンク型・リダイレクト型決済では、Webhookでシステムや他のSaaSと連携できる
- Webhook APIでは、注文処理からCRM・顧客サポートまで幅広い運用ができる
- ZapierやAWS StepFunctionsなどで、ノーコード・ローコードな組み込みも可能