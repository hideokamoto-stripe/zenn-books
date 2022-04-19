---
title: "Next.jsを利用したアプリのセットアップ"
---

まずはNext.jsでサイトをセットアップしましょう。

## なぜNext.jsか？

Next.jsを利用することで、フロントエンド・バックエンド両方のアプリをまとめて構築・管理することができます。

Vue.jsのNuxt.jsなど、他にも同様のフレームワークは存在しますが、今回はReactを利用するため、Next.jsをベースに進めます。

## Next.jsのセットアップ

早速セットアップを始めましょう。
ターミナルに以下のコマンドを入力して実行しましょう。

```bash
npx create-next-app@latest
```

すると「プロジェクト名を教えてください」という英語のメッセージが表示されます。
ここで`nextjs-stripe-ec`を入力し、[Enter]キーを押しましょう。

```
? What is your project named? › nextjs-stripe-ec
```

この後、1〜2分程度かけてプロジェクトのセットアップが始まります。
npmからライブラリのダウンロードを行いますので、ネットワークに接続した状態で待ちましょう。

```
[1/4] 🔍  Resolving packages...
[2/4] 🚚  Fetching packages...
[3/4] 🔗  Linking dependencies...
```

成功すると、以下のようにコマンドのガイドが表示されます。

```
Success! Created nextjs-stripe-ec at /Users/examples/nextjs-stripe-ec
Inside that directory, you can run several commands:

  npm run dev
    Starts the development server.

  npm run build
    Builds the app for production.

  npm run start
    Runs the built app in production mode.

We suggest that you begin by typing:

  cd nextjs-stripe-ec
  npm run dev
```

最後に、以下の２コマンドを実行して、作成したアプリを起動させてみましょう。

```bash
  cd nextjs-stripe-ec
  npm run dev
```

`npm run dev`の実行が成功すると、URLが表示されます。

```bash
ready - started server on 0.0.0.0:3000, url: http://localhost:3000
```

表示されているURLに、Chromeなどのブラウザからアクセスしましょう。

画像のようなサイトが表示されれば、セットアップ完了です。

![](https://storage.googleapis.com/zenn-user-upload/c9ab54f28690-20220418.png)

## ディレクトリ構造を見てみよう

セットアップしたプロジェクトは、おおよそ以下の様な構造です。

```bash
$ tree -I node_modules
.
├── README.md
├── next.config.js      // Next.jsの設定ファイル
├── package.json
├── pages               // ページ・APIを管理するディレクトリ
│   ├── _app.js         // 全てのページで実行するReact JSXファイル
│   ├── api             // REST APIのエンドポイント用のディレクトリ
│   │   └── hello.js    // REST API `/api/hello`で実行するファイル
│   └── index.js        // TOPページ用のReact JSXファイル
├── public
│   ├── favicon.ico     // デモ用のFavicon
│   └── vercel.svg      // デモ用の画像
└─── styles
    ├── Home.module.css // pages/index.jsで利用しているCSSファイル
    └── globals.css     // pages/_app.jsでimportしているCSSファイル
```

### 開発時の注意点
#### `pages/`ディレクトリ以下にComponent要素を配置しない
`pages/`にファイルを追加すると、そのファイル名でページを生成しようとします。
そのため、`pages/layout.jsx`の様に、レイアウトや機能目的のコンポーネントを追加すると、`http://localhost:3000/layout`でページが生成されてしまいます。

ページではないReactコンポーネントを追加する場合は、`components`ディレクトリを新しく作成し、そこに配置しましょう。
同様に、APIで利用する関数なども、`pages`や`pages/api`ではなく、`lib`ディレクトリを作成して配置しましょう。

#### ロゴなどの画像・静的ファイルは`public`ディレクトリへ

アイコン・ロゴ・Faviconなどの静的ファイルを追加したい場合、`public`ディレクトリ以下に配置しましょう。

そうすることで、`<img src="/logo.svg"/>`のように`/`から始まる相対パスでファイルを指定できます。

## ターミナルでの、作業場所の確認

ワークショップ中、何かのはずみで別のディレクトリに移動してしまい、コマンドが思った通りに動かないことがあります。
その場合に備えて、「いまどのディレクトリにいるか」を事前に確認しましょう。

まず、ターミナルで`pwd`と入力し、[Enter]キーを押しましょう。
すると、現在いるディレクトリが表示されます。

```bash
pwd
/Users/stripe/nextjs-stripe-ec
```

２行目に表示されている`/Users/~/nextjs-stripe-ec`をコピーして、メモ帳などに保存しておきましょう。

もしコマンドを実行した結果が、資料と異なっている場合、以下の手順でディレクトリの場所を確認できます。

- `pwd`コマンドを実行する
- 保存しておいた文字列と同じ結果かを確認する
- [同じだった場合]エラーの理由は別にありそうです。メッセージをGoogle翻訳などを使って読んでみましょう。
- [違った場合] `cd <コピーした文字列>`を実行しましょう。　例: `cd /Users/stripe/nextjs-stripe-ec`  
これでNext.jsアプリのルートに戻ってくることができます。
## おさらい

- Next.jsで、ReactとREST APIをまとめて構築・管理ができる
- `create-next-app`でNext.jsアプリをセットアップできる
- Next.jsでは、用途に応じてファイルの置き場所がある程度決まっている

次のステップでは、静的サイト（SSG mode）でアプリの構築を体験します。

### 「早く終わった」という方のための、もう１ステップ

`npx create-next-app --ts`で、TypeScriptを使えるようにしてみましょう。
[スターターアプリ](https://github.com/vercel/next.js/tree/canary/examples/with-typescript)や[ドキュメント](https://nextjs.org/docs/basic-features/typescript)で実装方法についてチェックすることができます。

もし余裕がある場合は、JavaScript版との違いを調べて、後付けでTypeScript対応させてみましょう。
Hint: https://nextjs.org/docs/basic-features/typescript#existing-projects