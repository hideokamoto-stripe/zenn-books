---
title: "Cloudflare Pages上で、テスト・本番環境を切り替える準備をしよう"
---

Stripeの組み込みが完了したアプリケーションを、Cloudflare Pagesにデプロイしましょう。

## 環境変数を、プレビュー・本番環境で設定しよう

まずは環境変数の設定を行います。

Stripeでは、テストモードと本番モードの2つがユーザーに提供されています。

そのため、本番環境としてデプロイするには、本番モードの購入ボタンやカスタマーポータルURLを設定する必要があります。

Cloudflareのダッシュボードに移動し、Pagesのプロジェクトを開きましょう。
![](https://storage.googleapis.com/zenn-user-upload/3c1712a7c720-20230624.png)

[設定]タブから、[環境変数]に移動します。
プロダクションとプレビューそれぞれに環境変数が設定できます。

![](https://storage.googleapis.com/zenn-user-upload/7a5ffade155e-20230624.png)

まずはプロダクション側で、[変数を追加する]ボタンをクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/b41a6a25edaa-20230624.png)

次の表やスクリーンショットを参考に、環境変数を３つ追加します。

|変数名|値|例|
|:--|:--|:--|
|`PUBLIC_STRIPE_PUBLISHABLE_API_KEY`|`stripe-buy-button`タグの、`publishable-key`の値|`pk_test_xxx`|
|`PUBLIC_STRIPE_BUY_BUTTON_ID`|`stripe-buy-button`タグの、`buy-button-id`の値|`buy_btn_xxxxx`|
|`PUBLIC_STRIPE_CUSTOMER_PORTAL_URL`|カスタマーポータルのURL|`https://billing.stripe.com/p/login/test_xxxx`|

![](https://storage.googleapis.com/zenn-user-upload/bb6f70f4727b-20230624.png)

保存後、プレビュー環境でも同じ値を登録しましょう。

![](https://storage.googleapis.com/zenn-user-upload/0ad64477a14f-20230624.png)

実運用では、プロダクション側にはStripeの本番環境で作成した購入ボタンやカスタマーポータルの値を設定します。


## Astro(SSRモード)の環境変数取得処理を追加しよう

環境変数の設定ができましたので、Astro側でもその値を読む準備をしましょう。

まずローカル環境で環境変数を読めるように、`.dev.vars`ファイルを作ります。

```env:.dev.vars
PUBLIC_STRIPE_BUY_BUTTON_ID=buy_btn_xxxx
PUBLIC_STRIPE_PUBLISHABLE_API_KEY=pk_test_xxxx
PUBLIC_STRIPE_CUSTOMER_PORTAL_URL=https://billing.stripe.com/p/login/test_xxxx
```

`.dev.vars`ファイルは、ignoreしておきましょう。

```diff env:.gitignore
# macOS-specific files
.DS_Store

+.dev.vars
```

その後、`src/pages/index.astro`を次のように変更しましょう。

```diff html:src/pages/index.astro
---
import Layout from '../layouts/Layout.astro';
import Card from '../components/Card.astro';
+import { getRuntime } from "@astrojs/cloudflare/runtime";
+const { env } = getRuntime(Astro.request);
---

<Layout title="Welcome to Astro.">
	<header>
-		<a href="https://billing.stripe.com/p/login/test_xxxxk">マイページ</a>
+		<a href={env.PUBLIC_STRIPE_CUSTOMER_PORTAL_URL}>マイページ</a>
	</header>
	<main>
	<main>
		<script async
		  src="https://js.stripe.com/v3/buy-button.js">
		</script>
		
		<stripe-buy-button
-		  buy-button-id="buy_btn_xxxxx"
+			buy-button-id={env.PUBLIC_STRIPE_BUY_BUTTON_ID}
-		  publishable-key="pk_test_xxxxxx"
+			publishable-key={env.PUBLIC_STRIPE_PUBLISHABLE_API_KEY}
		>
		</stripe-buy-button>
	</main>
</Layout>
```

最後に、ローカルで動作確認するコマンドを、Wrangler経由に変更します。

```diff json:package.json
  "scripts": {
    "dev": "astro dev",
    "start": "astro dev",
    "build": "astro build",
-    "preview": "astro preview",
+    "preview": "wrangler pages dev ./dist",
    "astro": "astro"
  },
```

最後に、次の２コマンドを実行しましょう。

```bash
% npm run build
% npm run preview
```

表示されたAstroアプリが、作業前と同じような表示になっていれば成功です。

:::message
`preview`では、コード変更の反映に都度`npm run build`コマンドの実行が必要です。
:::

最後に、Cloudflare Pagesにもデプロイしましょう。

```bash
% npm run build
% wrangler pages deploy dist 
```

デプロイ成功メッセージに、プレビューURLが含まれています。

```bash

✨ Success! Uploaded 0 files (2 already uploaded) (0.30 sec)

✨ Compiled Worker successfully
✨ Uploading Worker bundle
✨ Uploading _routes.json
✨ Deployment complete! Take a peek over at https://xxxx.yyy.pages.dev
✨  Done in 5.55s.
```

表示されている、https://xxxx.yyy.pages.dev にアクセスして、こちらも動作が問題ないか確認しましょう。

## Checkpoint

- 本番・テスト環境の切り替えは、環境変数で設定できる
- Cloudflare Pagesでは、プロダクション・プレビューで個別に環境変数が設定できる
- ローカルでは、`.dev.vars`ファイルで環境変数を設定し、`wrangler dev`コマンドで動かそう