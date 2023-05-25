---
title: "[Cloudflare & Stripe入門] Cloudflare WorkersとStripeで、オリジナルの決済リンクを作ろう"
emoji: "🚀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["cloudflare", "stripe", "javascript", "typescript"]
published: true
---

「Cloudflare Workers」を利用すると、サーバーレスなアプリケーションをCloudflareのデータセンターにデプロイできます。

この記事では、これからCloudflare Workers（以下Workers）を触ってみようという方向けの、簡単なAPIの作り方を紹介します。

## Stripe SDKを利用して、オリジナルのPayment Linksを作るAPIを作成しよう

ここでは、「Stripeの決済フォームへ遷移するAPI」をWorkers上に作りましょう。

- ユーザーがWorkersのURLにアクセスする
- Workers内でStripe SDKを利用して、Checkoutのセッションを作成する
- ユーザーを、作成したCheckoutのセッションURLにリダイレクトする
- リダイレクト先で決済処理を行う

Stripe Payment Links・・・のようなものを自作するイメージです。

### Step1: Wranglerを利用して、Workersプロジェクトをセットアップする

まずはローカル環境とデプロイの準備をするために、[Wrangler](https://developers.cloudflare.com/workers/wrangler/)をインストールします。

```bash
# Cloudflareアカウントにログイン
% npx wrangler login

# プロジェクトのディレクトリを作成
% mkdir cloudflare-stripe
% cd cloudflare-stripe

# WranglerでWorkersプロジェクトを立ち上げ
% npx wrangler init
```

対話形式で、「TypeScriptを使うか？」「テストコードは必要か？」などを訊かれます。

今回はAPIを作成しますので、`Would you like to create a Worker at src/index.ts?`では`Fetch handler`を選択しましょう。

```bash
? Would you like to create a Worker at src/index.ts? › - Use arrow-keys. Return to submit.
    None
❯   Fetch handler
    Scheduled handler
```

それ以外の設定については、次のように設定しています。

```bash
 ⛅️ wrangler 2.12.0 (update available 2.13.0)
-------------------------------------------------------
Using npm as package manager.
✨ Created wrangler.toml
✔ Would you like to use git to manage this Worker? … yes
✨ Initialized git repository
✔ No package.json found. Would you like to create one? … yes
✨ Created package.json
✔ Would you like to use TypeScript? … yes
✨ Created tsconfig.json
✔ Would you like to create a Worker at src/index.ts? › Fetch handler
✨ Created src/index.ts
✔ Would you like us to write your first test with Vitest? … yes
✨ Created src/index.test.ts
```

`src/index.ts`または`src/index.js`が作成されていれば、セットアップ完了です。

`npx wrangler dev`を実行すると、ローカル環境でAPIが立ち上がります。

```bash
 ⛅️ wrangler 2.15.0 (update available 2.15.1)
-------------------------------------------------------
⬣ Listening at http://0.0.0.0:60376
- http://127.0.0.1:60376
- http://192.168.86.23:60376
- http://172.18.108.81:60376
Total Upload: 0.19 KiB / gzip: 0.16 KiB
```

### Step2: Stripe SDKを利用して、ローカル環境でAPIを開発する

APIの準備ができましたので、StripeのAPIを呼び出す処理を追加しましょう。

#### APIキーを取得し、環境変数に設定しよう

Stripeダッシュボードから、[シークレットAPIキー]または[制限付きAPIキー]を取得しましょう。

![](https://storage.googleapis.com/zenn-user-upload/c5a6cba35670-20230420.png)

取得したキーを、`.dev.vars`ファイルに保存します。

```
STRIPE_SECRET_API_KEY=sk_test_xxxx
```

#### Stripe SDKをインストールし、APIキーを設定する

StripeのJS SDKをnpmでインストールしましょう。

```bash
% npm i stripe
```

Workersでは、Node.jsのAPIが一部サポートされていません。

ですが、StripeのJS SDKはWorkersでも動作するようにアップデートされていますので、問題なく動作します。

https://blog.cloudflare.com/ja-jp/announcing-stripe-support-in-workers-ja-jp/

環境変数に設定したAPIキーを利用して、Stripe SDKをセットアップしましょう。

```diff typescript
+import Stripe from 'stripe'

export default {
	async fetch(request, env, ctx) {
+		const stripe = new Stripe(env.STRIPE_SECRET_API_KEY, {
+			apiVersion: '2022-11-15'
+		})
...
```

`process.env`ではなく、`env`から取得する点に注意です。

APIキーを読み込みするため、念の為`npx wrangler dev`コマンドは再起動しましょう。

:::message
TypeScriptで開発する場合、`src/index.ts`の`interface Env`を次のように編集します。

```diff typescript
export interface Env {
+	STRIPE_SECRET_API_KEY: string;
}
```

これで`env.STRIPE_SECRET_API_KEY`で型エラーが出なくなります。
:::


#### サブスクリプション商品と申込のセッションを作成しよう

APIを呼び出す準備ができましたので、決済・申込のためのセッションを作成しましょう。

まずは商品データを作成します。

```diff typescript

export default {
	async fetch(request, env, ctx) {
		const stripe = new Stripe(env.STRIPE_SECRET_API_KEY, {
			apiVersion: '2022-11-15'
		})

+		const product = await stripe.products.create({
+			name: 'Cloudflare Workers デモ'
+		})

```

その後、作成した商品データを利用して、申込のセッションを作成します。

```diff typescript

export default {
	async fetch(request, env, ctx) {
		const stripe = new Stripe(env.STRIPE_SECRET_API_KEY, {
			apiVersion: '2022-11-15'
		})

		const product = await stripe.products.create({
			name: 'Cloudflare Workers デモ'
		})
+		const session = await stripe.checkout.sessions.create({
+			success_url: 'https://example.com/thanks',
+			cancel_url: 'https://example.com',
+			line_items: [{
+				price_data: {
+					unit_amount: 1000,
+					currency: 'jpy',
+					recurring: {
+						interval: 'month',
+					},
+					product: product.id
+				},
+				quantity: 1,
+				adjustable_quantity: {
+					enabled: true,
+					maximum: 10,
+					minimum: 1,
+				}
+			}],
+			mode: 'subscription',
+		})
```

最後に決済ページへのリダイレクトを指示しましょう。

```diff typescript

		const session = await stripe.checkout.sessions.create({
			success_url: 'https://example.com/thanks',
			cancel_url: 'https://example.com',
			line_items: [{
				price_data: {
					unit_amount: 1000,
					currency: 'jpy',
					recurring: {
						interval: 'month',
					},
					product: product.id
				},
				quantity: 1,
				adjustable_quantity: {
					enabled: true,
					maximum: 10,
					minimum: 1,
				}
			}],
			mode: 'subscription',
		})
+		if (!session.url) {
+			const errorResponse = new Response("Failed to get the checkout url", {
+				status: 500,
+			})
+			return errorResponse
+		}
+		return Response.redirect(session.url, 301)
```

これで`npx wrangler dev`で表示されているURLにアクセスすると、Stripeのサブスクリプション申込ページにリダイレクトされます。

![](https://storage.googleapis.com/zenn-user-upload/d097fdd51ab1-20230420.png)


### Step3: Cloudflare上に、コードとAPIキーをデプロイする

最後に作成したコードをデプロイしましょう。

#### Wranglerで、StripeのAPIキーをCloudflareアカウントにアップロードする

APIキーなどのシークレット情報は、`npx wrangler secret put <キー名>`でアップロードできます。

`.dev.vars`で設定した環境変数と同じ名前でアップロードしましょう。

```bash
% npx wrangler secret put STRIPE_SECRET_API_KEY
-------------------------------------------------------
? Enter a secret value: › 
```

`Enter a secret value`で、`sk_`または`rk_`から始まるAPIキーをコピーアンドペーストしましょう。

完了メッセージが表示されれば、成功です。

```bash
✔ Enter a secret value: … ***********************************************************************************************************
🌀 Creating the secret for the Worker "cloudflare-stripe" 
✨ Success! Uploaded secret STRIPE_SECRET_API_KEY
```

#### Wranglerで、Workersのコードをデプロイする

デプロイもWranglerで実施できます。

`npx wrangler publish`を実行しましょう。

```bash
% npx wrangler publish
 ⛅️ wrangler 2.15.1
-------------------------------------------------------
Total Upload: 188.66 KiB / gzip: 36.73 KiB
Uploaded pico (1.95 sec)
```

デプロイに成功すると、WorkersのURLが表示されます。

```bash
Published pico (4.25 sec)
  https://cloudflare-stripe.YOUR_WS_NAME.workers.dev
```

URLにアクセスして、Stripeの決済ページが表示されれば成功です。



## 実践的なカスタマイズ方法の例

Stripeを利用した、簡単なWorkersでのAPI開発方法を紹介しました。

ユースケースとしては、以下のような場面が考えられます。

### 1: Payment Linksでサポートされていない設定を持つURLを発行する

「Payment Linksでは利用できないけども、Checkoutでは利用できる設定」は複数存在します。

そこでWorkersなどを利用してURLを発行することで、Payment Linksのワークアラウンドが実現できます。

```diff

+		const now = new Date();
+		const hourLater = new Date(now.getTime() + (60 * 60 * 1000));
		const session = await stripe.checkout.sessions.create({
			success_url: 'https://example.com/thanks',
			cancel_url: request.headers.get("Referer") || 'https://example.com',
			line_items: [{
				price: 'price_1xxxxx',
				quantity: 1,
				adjustable_quantity: {
					enabled: true,
					maximum: 10,
					minimum: 1,
				}
			}],
			mode: 'payment',
+			expires_at: Math.floor(hourLater.getTime()/ 1000)
		})
```

このサンプルでは、「URLアクセスから、1時間以内に完了させる必要のある決済URL」を作成しています。

### 2: 販売期限付きの販売URLを発行する

Workers側で日時の判定処理を行うことで、「指定した日時を過ぎると、無効化されるURL」を発行できます。

```diff
export default {
	async fetch(
		request: Request,
		env: Env,
		ctx: ExecutionContext
	): Promise<Response> {
+		const now = new Date();
+		if (now.getTime() > new Date('2023/04/01 00:00').getTime()) {
+			return new Response("Not found", {
+				status: 404,
+			})
+		}
		...
```

このサンプルコードでは、2023/04/01 00:00を過ぎてアクセスした場合、404エラーを返します。

販売期間の開始日と終了日を環境変数にすることで、不定期に販売を行う商品のURL発行などにも利用できます。

```diff
export default {
	async fetch(
		request: Request,
		env: Env,
		ctx: ExecutionContext
	): Promise<Response> {
		const now = new Date();
		if (
			now.getTime() < new Date(env.SALE_START_DATE).getTime() ||
			now.getTime() > new Date(env.SALE_END_DATE).getTime()
		) {
			return new Response("Not found", {
				status: 404,
			})
		}
```

**.dev.vars**

```
STRIPE_SECRET_API_KEY=sk_test_xxxxxx
SALE_START_DATE=2023/04/01
SALE_END_DATE=2023/04/15
```

### 3: 外部の認証情報を利用した、既存顧客向けのステートフルURLを発行する

Workers内で、Okta CIC(Auth0)やfirebaseなどのAPIを利用したユーザー認証を行う方法です。

これによって、指定されたHeader情報を含むアクセス以外では、決済ページに遷移できないURLが作れます。

```diff
export default {
	async fetch(
		request: Request,
		env: Env,
		ctx: ExecutionContext
	): Promise<Response> {
+		const BASIC_USER = "admin"
+		const BASIC_PASS = "admin"
+		const Authorization = request.headers.get("Authorization")
+		if (!Authorization) {
+			return new Response("You need to login.", {
+			  status: 401,
+			  headers: {
+				"WWW-Authenticate": 'Basic realm="my scope", charset="UTF-8"',
+			  },
+			})
+		}
+		const [scheme, encoded] = Authorization.split(' ')
+		if (!encoded || scheme !== 'Basic') {
+			return new Response("Unauthorized", {
+				status: 400,
+				statusText: 'Bad Request',
+			})
+		}
+		const buffer = Uint8Array.from(
+			atob(encoded),
+			(character) => character.charCodeAt(0)
+		)
+		const decoded = new TextDecoder().decode(buffer).normalize()
+ 
+		const index = decoded.indexOf(":")
+		if (index === -1 || /[\0-\x1F\x7F]/.test(decoded)) {
+			return new Response("Invalid authorization value", {
+				status: 400,
+				statusText: 'Bad Request',
+			})
+		}
+		const user = decoded.substring(0, index)
+		const pass = decoded.substring(index + 1)
+		if (user !== BASIC_USER || pass !== BASIC_PASS) {
+			return new Response("Invalid credentials", {
+				status: 401,
+				statusText: 'Unauthorized',
+			})
+		}

```

このサンプルは、Cloudflareが提供するBASIC認証のサンプルコードを簡易化したものです。

https://developers.cloudflare.com/workers/examples/basic-auth/

## 終わりに

簡単なサンプルアプリケーションの例として、Stripe Checkoutを利用した決済URLの作り方を紹介しました。

この他にも様々なユースケースでCloudflare WorkersやStripeを活用できます。

ぜひあなたの活用ユースケースを、[#JP_Stripesハッシュタグ](https://twitter.com/hashtag/JP_Stripes)などで、Stripeコミュニティに共有してください💪

## 参考記事・ドキュメント

https://developers.cloudflare.com/workers/examples/basic-auth/

https://developers.cloudflare.com/workers/examples/redirect/

https://blog.cloudflare.com/ja-jp/announcing-stripe-support-in-workers-ja-jp/
