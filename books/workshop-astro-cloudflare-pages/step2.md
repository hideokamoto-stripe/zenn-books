---
title: "Astroで作成したアプリを、Cloudflare Pagesへデプロイしよう"
---

Astro作成したアプリを、Cloudflareで公開してみましょう。

## Wranglerをインストールしよう

ローカル環境から直接デプロイするには、[Wrangler](https://developers.cloudflare.com/workers/wrangler/)を利用します。

https://blog.cloudflare.com/ja-jp/wrangler-v2-beta-ja-jp/

インストールは、`npm`で実行します。

```bash
% npm install -g wrangler
```

:::message
グローバルインストールしたくない場合は、`npm i -D wrangler`でインストールしましょう。
その後、`wrangler`コマンドを`npx wrangler`に読み替えて続けます。
:::

WranglerがCloudflareアカウントにアクセスするには、OAuth2.0での認証が必要です。

ログインコマンドを実行しましょう。

```bash
% wrangler login
```

ブラウザが立ち上がり、CLIからのアクセス許可を求める画面が開きます。

![](https://storage.googleapis.com/zenn-user-upload/4bb94e77c9d5-20230421.png)

もしブラウザが立ち上がらないか、タブを見失った場合には、`wrangler login`を実行したターミナルにURLが表示されています。これをクリックしてアクセスしましょう。

```
-------------------------------------------------------
Attempting to login via OAuth...
Opening a link in your default browser: https://dash.cloudflare.com/oauth2/auth?response_type=code&XXXX&
```

`Allow`ボタンを押すと、成功画面に切り替わります。

![](https://storage.googleapis.com/zenn-user-upload/6606581ba373-20230421.png)

`wrangler login`を実行したターミナル画面にも、成功メッセージが表示されます。

```bash
Successfully logged in.
? Would you like to help improve Wrangler by sending usage metrics to Cloudflare? › (Y/n)

Your choice has been saved in the following file: ../../.wrangler/metrics.json.

  You can override the user level setting for a project in `wrangler.toml`:

   - to disable sending metrics for a project: `send_metrics = false`
   - to enable sending metrics for a project: `send_metrics = true`
```

これで手元のPCからCloudflareアカウントにアクセスできるようになりました。

## WranglerでAstroサイトをデプロイする

早速アプリをCloudflare Pagesへデプロイしましょう。

デプロイの前に、Astroでアプリをビルドします。

```bash
% npm run build
```

その後、`wrangler pages`コマンドでCloudflare Pagesへデプロイしましょう。

```bash
% wrangler pages deploy dist
```

Cloudflare Pagesに新しくプロジェクトを作成するか聞かれます。
`Create a new project`を選びましょう。

```bash
No project selected. Would you like to create one or use an existing project?
❯ Create a new project
  Use an existing project
```

プロジェクト名を入力します。
Cloudflare Pagesの公開ドメイン名に利用されるため、`<yourname>-jpstripes-cf-ws`など、他にこのワークショップに取り組んでいる方と重複しない名前を設定しましょう。

```bash
? Enter the name of your new project: › 
```

Gitリポジトリのブランチを訊かれます。
現在のブランチ名がデフォルトの選択肢として表示されますので、そのまま[Enter]キーを押しましょう。

```bash
? Enter the production branch name: › main
```

`dist`ディレクトリにある、`npm run build`で作成したファイルがアップロードされます。

```bash
✨ Successfully created the 'ws-stripe-cloudflare-astro' project.
▲ [WARNING] Warning: Your working directory is a git repo and has uncommitted changes

  To silence this warning, pass in --commit-dirty=true


🌏  Uploading... (3/3)

```

最後に成功メッセージが表示されます。

```bash
✨ Success! Uploaded 3 files (3.09 sec)

✨ Deployment complete! Take a peek over at https://xxxxx.ws-stripe-cloudflare-astro.pages.dev
```

Cloudflareの管理画面にも、作成したPagesプロジェクトが表示されます。


![](https://storage.googleapis.com/zenn-user-upload/2cba2360149e-20230418.png)


この状態で、 `https://<設定したCloudflare Pagesプロジェクト名>.pages.dev`にアクセスすると、Astroで作成したページが表示されます。

![](https://storage.googleapis.com/zenn-user-upload/6553e32c543a-20230421.png)

これでAstroを利用した、**静的なウェブサイト**のデプロイは完了です。

## サーバー側を追加してみよう

Astroでは、サーバー側の処理をページ表示時に実行できます。

`src/pages/index.astro`を次のように変更しましょう。

```diff html:src/pages/index.astro

+const now = new Date()
---

<Layout title="Welcome to Astro.">
	<main>
+		<h1>現在<span class="text-gradient">{`${now.getHours()}時${now.getMinutes()}分${now.getSeconds()}秒`}</span>です</h1>
-		<h1>Welcome to <span class="text-gradient">Astro</span></h1>
```

変更を保存して、Cloudflare Pagesにデプロイします。

```bash
% npm run build
% wrangler pages publish dist
```

すると、Cloudflare PagesのURLに何回アクセスしても、同じ分秒が表示され続けます。

![](https://storage.googleapis.com/zenn-user-upload/12b95ea66485-20230421.png)

これは、`npm run build`でのビルド時に静的化されているためです。

サーバー側の処理を実行したい場合は、設定を変更する必要があります。

## Cloudflare PagesでAstroのSSRに対応する

Astroでは、デプロイ先のサービスによってSSRを有効化する方法が異なります。

`astro`コマンドでCloudflare用のモジュールを追加しましょう。

```bash
% npx astro add cloudflare
```

`@astrojs/cloudflare`のインストールを求められますので、`y`と回答してインストールしましょう。

```bash
  Astro will run the following command:
  If you skip this step, you can always run it yourself later

 ╭────────────────────────────────────────────╮
 │ npm i @astrojs/cloudflare astro@^2.1.4     │
 ╰────────────────────────────────────────────╯

? Continue? › (Y/n)
```

続いて設定ファイル（`astro.config.mjs`）の変更も求められます。
こちらも`y`で続行しましょう。

```
  Astro will make the following changes to your config file:

 ╭ astro.config.mjs ──────────────────────────────╮
 │ import { defineConfig } from 'astro/config';   │
 │                                                │
 │ import cloudflare from "@astrojs/cloudflare";  │
 │                                                │
 │ // https://astro.build/config                  │
 │ export default defineConfig({                  │
 │   output: "server",                            │
 │   adapter: cloudflare()                        │
 │ });                                            │
 ╰────────────────────────────────────────────────╯

  For complete deployment options, visit
  https://docs.astro.build/en/guides/deploy/

? Continue? › (Y/n)
```

次のメッセージが表示されれば、設定完了です。

```
✔ Continue? … yes
  
   success  Added the following integration to your project:
  - @astrojs/cloudflare
```

ここまで完了した状態で、もう一度ビルドとデプロイを実行しましょう。

```bash
% npm run build
% wrangler pages publish dist
```

今度はページにアクセスするたびに、秒数や分数が変わります。

![](https://storage.googleapis.com/zenn-user-upload/dc8039b27940-20230421.png)

## おさらい

- Wranglerを使って、Cloudflare Pagesにサイトをデプロイできる
- AstroでSSRを使う場合、デプロイ先によって設定が変わる
- Cloudflare Pagesでも、AstroやNext.jsのSSRが利用できる

次のステップからは、Stripeを利用したサブスクリプション申し込みフローを実装します。