---
title: "[Advanced] Cloudflare KVで、StripeとAstroを連携させてみよう"
---

もし想定していた時間よりも早く作業が終わった場合、こちらの章をお試しください。

ここでは、申し込みしたサブスクリプションの顧客IDをアプリ内で再利用できるようにするため、Cloudflare KVやStripe Webhookが登場します。

## ローカル環境で、AstroアプリとKVを接続する

まずは手元の環境で、KVを動かしてみましょう。

いくつかのファイルが作成されますので、`.wrangler`ディレクトリをignoreしておきます。

```diff env:.gitignore
# macOS-specific files
.DS_Store

.dev.vars
+.wrangler/
```

ローカルでKVを試す場合は、`--kv`オプションを追加しましょう。

```bash
wrangler pages dev dist/ --kv WSDEMO
```

これでローカル環境でKVを利用する準備ができました。

## StripeのシークレットAPIキーを取得する

Stripeのデータを取得するには、APIキーを利用します。
APIキーを確認するには、ダッシュボードの[開発者]メニューを開きましょう。

![](https://storage.googleapis.com/zenn-user-upload/95097c74935d-20220419.png)

続いて、左側のメニューから[APIキー]をクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/6fce96bb18c1-20220419.png)

:::message
Tips: Stripe 3つのAPIキー

APIキーには3つのAPIキーが存在します。

- 1つ目は「公開可能キー」です。これはフロントエンドアプリで、カードのtoken化などに利用します。
- 2つ目は「シークレットキー」です。こちらはStripeの公開しているAPI全てにアクセスができるAPIで、万が一漏洩した場合には顧客情報や注文データなどを取得されるリスクが存在します。
- 最後のキーは、「制限付きのキー」です。シークレットキー同様にStripeのAPIにアクセスするために利用しますが、アクセスできるリソースや操作を個別に設定することができます。

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


### Stripe APIキーを、ローカルとPagesの環境変数に設定しよう

取得したシークレットをローカルとデプロイ先で利用できるようにします。

ローカルでは、`.dev.vars`ファイルに追加しましょう。

```diff env:.dev.vars
PUBLIC_STRIPE_CUSTOMER_PORTAL_URL=https://billing.stripe.com/p/login/xxxx
+STRIPE_API_KEY=rk_test_xxxxx
```

Cloudflare Pagesには、環境変数から登録します。

![](https://storage.googleapis.com/zenn-user-upload/f3e431f261ae-20230624.png)


プレビューを利用する場合は、そちらにも忘れずに登録しましょう。


## StripeのWebhookシークレットを取得する

KVに保存するデータは、Stripe上で新しく顧客情報が作成された際に送信されるWebhookデータを利用します。

Webhook APIを追加する前に、「Stripeからのリクエストかどうか」を検証するシークレットを取得しましょう。

Cloudflare Pagesの詳細ページTOPにある、[ドメイン]のURLをコピーします。

![](https://storage.googleapis.com/zenn-user-upload/4a494c96b654-20230624.png)

Stripeダッシュボードを開いて、[開発者]タブから[Webhook]を選びます。
![](https://storage.googleapis.com/zenn-user-upload/05139b7d0ab4-20230624.png)

Webhookの連携設定画面が開きます。
[リッスンするイベントの選択]をクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/879d03c3246c-20230624.png)

どのようなイベントをAPIに送信するかを選ぶ画面が開きます。

![](https://storage.googleapis.com/zenn-user-upload/529f785dc18a-20230624.png)

[Customer]から、[customer.created]にチェックを入れましょう。


![](https://storage.googleapis.com/zenn-user-upload/afe4540fdb28-20230624.png)

[イベントを追加]ボタンをクリックすると、設定が更新されます。

![](https://storage.googleapis.com/zenn-user-upload/e3b4deab1a63-20230624.png)

以下の表を参考に、残りの情報を入力しましょう。


|項目名|入力する値|
|:--|:--|
|エンドポイントURL|https://<Cloudflare PagesのURL>/webhook|
| リッスンする | アカウントでのイベント |
| バージョン | 最新のAPIバージョン|
| リッスンするイベントの選択|`customer.created`|

最後に[イベントを追加]ボタンをクリックして、保存します。


![](https://storage.googleapis.com/zenn-user-upload/06cbd4307ba9-20230624.png)

[署名シークレット]セクションの下部にある[表示]をクリックすると、署名シークレットが表示されます。

これをコピーしましょう。

![](https://storage.googleapis.com/zenn-user-upload/ee5fb47eb659-20230624.png)

### 署名シークレットを、ローカルとPagesの環境変数に設定しよう

取得したシークレットをローカルとデプロイ先で利用できるようにします。

ローカルでは、`.dev.vars`ファイルに追加しましょう。

```diff env:.dev.vars
PUBLIC_STRIPE_CUSTOMER_PORTAL_URL=https://billing.stripe.com/p/login/xxxx
STRIPE_API_KEY=rk_test_xxxxx
+STRIPE_WEBHOOK_SECRET=whsecxxxx
```

Cloudflare Pagesには、環境変数から登録します。

![](https://storage.googleapis.com/zenn-user-upload/ca005bbbe6ed-20230624.png)

プレビューを利用する場合は、そちらにも忘れずに登録しましょう。

![](https://storage.googleapis.com/zenn-user-upload/e51eb9ff0bcc-20230624.png)

## Astroで、Webhook APIを作成する

環境変数などの準備ができましたので、APIを作成しましょう。

まずは署名シークレットを利用したリクエストの検証処理を追加します。

StripeのSDKを利用しますので、インストールしましょう。

```bash
npm i stripe
```

続いて、`src/pages/webhook.ts`を作成しましょう。

```typescript:src/pages/webhook.ts
import type { APIRoute } from "astro";
import Stripe from 'stripe';
import { getRuntime } from "@astrojs/cloudflare/runtime";

export const post: APIRoute = async ({ request }) => {
    const runtime = getRuntime(request);
    const stripe = new Stripe((runtime.env as any).STRIPE_API_KEY, {
        apiVersion: "2022-11-15",
        httpClient: Stripe.createFetchHttpClient(), 
    });
    const signature = request.headers.get("stripe-signature");
    try {
        if (!signature) {
            throw new Error("signature is invalid.");
        }
        const body = await request.text();
        const event = await stripe.webhooks.constructEventAsync(
            body,
            signature,
            (runtime.env as any).STRIPE_WEBHOOK_SECRET,
            undefined,
            Stripe.createSubtleCryptoProvider();
        );
        if (event.type === 'customer.created') {
            await (runtime.env as any).WSDEMO.put('customer_id', (event.data.object as Stripe.Customer).id)
        }
        return {
            body: JSON.stringify({
                message: "ok",
            })
        };
      } catch (err) {
        const errorMessage = `⚠️  Webhook signature verification failed. ${err instanceof Error ? err.message : 'Internal server error'}`
        console.log(errorMessage);
        return new Response(errorMessage, {
            status: 400
        })
      }
}
```

その後、KVにデータを保存する処理を追加しましょう。

```diff typescript:src/pages/webhook.ts
        const event = stripe.webhooks.constructEvent(
            Buffer.from(body),
            signature,
            (runtime.env as any).STRIPE_WEBHOOK_SECRET
        );

+        if (event.type === 'customer.created') {
+            await (runtime.env as any).WSDEMO.put('customer_id', (event.data.object as Stripe.Customer).id)
+        }

        return {
            body: JSON.stringify({
                message: "ok",
            })
        };
```

これで、Stripe上に新しく顧客データが作成されるたびに、KVの`customer_id`に作成された顧客のIDが保存されます。

## Astroで、KVのデータを利用した「顧客マイページ」を作ろう

ノーコードで紹介したカスタマーポータルですが、[Stripeの顧客ID(customer_id)]がわかれば、メール認証をスキップできます。

リダイレクトが発生するため、こちらもAPIとして作成しましょう。

`src/pages/mypage.ts`を作成し、次のコードを追加します。

```typescript:src/pages/mypage.ts
import type { APIRoute } from "astro";
import Stripe from 'stripe';
import { getRuntime } from "@astrojs/cloudflare/runtime";


export const get: APIRoute = async (context) => {
    const runtime = getRuntime(context.request);
    const stripe = new Stripe((runtime.env as any).STRIPE_API_KEY, {
        apiVersion: "2022-11-15"
    });
    const customerId = await (runtime.env as any).WSDEMO.get('customer_id')
    if (!customerId) {
        return context.redirect((runtime.env as any).PUBLIC_STRIPE_CUSTOMER_PORTAL_URL, 302)
    }
    const customerPortal = await stripe.billingPortal.sessions.create({
        customer: customerId
    })
    return context.redirect(customerPortal.url, 302)
}
```

その後、カスタマーポータルに移動するリンクを、作成したAPIのパスに変更します。

```diff html:src/pages/index.astro

	<header>
-		<a href={env.PUBLIC_STRIPE_CUSTOMER_PORTAL_URL}>マイページ</a>
+		<a href="/mypage">マイページ</a>
	</header>
```


## Cloudflareダッシュボードで、KVを作成する

デプロイする前に、Cloudflare上で利用するKVを作成しましょう。

Cloudflareダッシュボードの[Workers & Pages]の、[KV]を開きます。

![](https://storage.googleapis.com/zenn-user-upload/702776110755-20230624.png)

[名前空間を作成する]をクリックして、作成ページに移動しましょう。

作成する名前空間は、ローカル環境と同じものを設定する必要があります。

今回のワークショップでは、`WSDEMO`を利用しましょう。

![](https://storage.googleapis.com/zenn-user-upload/d7affd69e7ca-20230624.png)

[追加]を押して作成した後は、デプロイ先のPagesプロジェクトに移動します。

[設定]から[Functions]に移動してください。

![](https://storage.googleapis.com/zenn-user-upload/924f14d580ff-20230624.png)

スクロールすると、[KV名前空間のバインディング]セクションが表示されます。

[バインディングを追加]をクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/04aa0f00ed86-20230624.png)

[変数名]と[KV名前空間]どちらも、`WSDEMO`で設定します。

![](https://storage.googleapis.com/zenn-user-upload/b23019013ce5-20230624.png)

[保存]を押して表示が変われば、OKです。

![](https://storage.googleapis.com/zenn-user-upload/b250d3736d13-20230624.png)

プレビューを利用する場合は、こちらにも同じ設定を行いましょう。

## Cloudflare PagesのFunctionを「互換性モード」にしてデプロイする

署名シークレットの検証には、`Node.js`のAPIが必要です。

そのため、ダッシュボードで「互換性モード」を有効化しましょう。

[設定]から[Functions]に移動してください。

![](https://storage.googleapis.com/zenn-user-upload/4e120d3e28a7-20230624.png)

スクロールして表示される[互換性フラグ]に`nodejs_compat`を設定します。

![](https://storage.googleapis.com/zenn-user-upload/8b8dc4667e38-20230624.png)

デプロイを実行して変更をアプリに反映させましょう。

```bash
% npm run build
% wrangler pages deploy ./dist
```

## 動作を確認してみよう

つなぎ込みの準備ができましたので、動作確認をはじめましょう。

[購入ボタン]のリンクをクリックして、サブスクリプションの申し込みを行います。

![](https://storage.googleapis.com/zenn-user-upload/e8e29a54f9aa-20230623.png)

今回は、メールによる認証を行いませんので、`test@example.com`などのダミーのアドレスで問題ありません。

|項目|入力値|備考|
|:--|:--|:--|
|メールアドレス|`test@example.com`||
|カード番号|4242424242424242||
|有効期限|12(月) / 40(年)|未来の月と年を入力しましょう。|
|セキュリティコード|111|テストモードでは、３桁の数字ならなんでもOKです。|
|カード所有者名|<あなたのお名前>||
|「情報を〜」|オフのまま||

入力が完了したら、[申し込む]ボタンをクリックします。

![](https://storage.googleapis.com/zenn-user-upload/ba0670a52c1d-20230623.png)

申し込みが完了したら、Astroアプリのトップページに戻りましょう。


![](https://storage.googleapis.com/zenn-user-upload/68e6ecad8d60-20230623.png)

カスタマーポータルへのリンクをクリックして、いきなりマイページに移動できれば、成功です。

![](https://storage.googleapis.com/zenn-user-upload/dfc59bd91a3d-20230623.png)

:::message
今回のデモでは、どのブラウザからアクセスしても同じ顧客のマイページに移動します。
実際のアプリケーション開発では、SupabaseやOkta CICなどの認証系サービスを併せてご利用ください。
:::


## [tips]カスタマーポータルの部分利用

「カード情報の更新画面だけ、カスタマーポータルを利用したい」のようなユースケースにも対応できます。

カスタマーポータルのURLを作成する処理に、`flow_data`を追加しましょう。

```diff typescript:src/pages/mypage.ts
    const customerPortal = await stripe.billingPortal.sessions.create({
        customer: customerId,
+        flow_data: {
+            type: 'payment_method_update',
+        },
    })
    return context.redirect(customerPortal.url, 302)

```

これをデプロイすると、「決済情報の更新画面だけのページ」にリダイレクトされます。

![](https://storage.googleapis.com/zenn-user-upload/5335cbb9b380-20230624.png)

様々なカスタマイズ方法が用意されていますので、詳細はDocsをご確認ください。

https://stripe.com/docs/customer-management/portal-deep-links


## Checkpoint

- PagesとWorkersでは、KVなどの接続（バインディング）方法が異なるので注意
- Webhook連携をする際は、必ず署名シークレットなどでリクエストの検証を！
- カスタマーポータルを使いこなして、サブスクリプションの運用業務を効率化しよう
