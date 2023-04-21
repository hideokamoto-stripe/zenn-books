---
title: "Astroを利用したアプリのセットアップ"
---

まずはサブスクリプション申し込みページを作るためのサイトを用意しましょう。

ここでは、AstroというJSフレームワークを利用します。

https://astro.build/

## Astroとは？

Astroは、ウェブサイトなどを高速で配信するための、オールインワン型JSフレームワークです。

ReactにおけるNext.jsや、VueにおけるNuxtのようなものだと考えて頂ければよいかなと思います。

UIフレームワークに依存せず利用できるため、ReactやVue, SvelteなどのUIフレームワークをAstroに持ち込む・併用することも可能です。

また、CloudflareやDenoなどのエッジコンピューティングでの配信に対応しています。

## Astroでアプリをセットアップしよう

Astroで新しくアプリを立ち上げるには、`npm create astro@latest`コマンドを利用します。

ターミナルで`npm create astro@latest`コマンドを実行しましょう。

すると、[HusonAI](https://houston.astro.build/)というGPT-3ベースのAIが、アプリの設定についていくつか質問してきます。

```bash
╭─────╮  Houston:
│ ◠ ◡ ◠  Keeping the internet weird since 2021.
╰─────╯

 astro   v2.3.0 Launch sequence initiated.

```

### アプリのディレクトリ名を指定する

まずはじめに、アプリのディレクトリ名を決める必要があります。

```bash
   dir   Where should we create your new project? 
```

ここでは、`ws-stripe-cloudflare-astro`と入力しましょう。

```bash
   dir   Where should we create your new project?
         ws-stripe-cloudflare-astro 
```

### テンプレートを選択する

ユースケースなどを簡単に学ぶことができるように、テンプレートが用意されています。

```
  tmpl   How would you like to start your new project?
         ● Include sample files (recommended)
         ○ Use blog template 
         ○ Empty 
```

ここでは、`Include sample files (recommended)`を選びましょう。

[Enter]キーを入力すると、ファイルのコピーが始まります。

```bash
 ██████  Template copying...
```

テンプレートの`package.json`に記載されたライブラリを追加するか聞かれます。
`Yes`を選択して、インストールを始めましょう。

```bash
  deps   Install dependencies? (recommended)
         ● Yes  ○ No 
```

### TypeScriptの設定を行う

インストールが完了すると、TypeScriptで開発するか否かを聞かれます。

```bash
   ts   Do you plan to write TypeScript?
         ● Yes  ○ No 
```

`Yes`を選択し、続いて`tsconfig.json`の設定をプリセットから選びます。

```bash
   use   How strict should TypeScript be?
❯   Strict - (recommended)
    Strictest
    Relaxed
    Help me choose
```

資料作成時には`Strict`を選択しましたが、それ以外でも進行上問題ありません。

### Gitリポジトリを作成しよう

最後にGitリポジトリを作成しましょう。

```bash
   git   Initialize a new git repository? (optional)
         ● Yes  ○ No 
```

### アプリのディレクトリに移動しよう

次のようなメッセージが表示されれば、セットアップ完了です。

```bash
 Enter your project directory using cd ./ws-stripe-cloudflare-astro 
 Run npm run dev to start the dev server. CTRL+C to stop.
 Add frameworks like react or tailwind using astro add.

 Stuck? Join us at https://astro.build/chat

╭─────╮  Houston:
│ ◠ ◡ ◠  Good luck out there, astronaut! 🚀
╰─────╯
```

次のコマンドを実行すると、サンプルアプリが立ち上がります。

```bash
% cd ws-stripe-cloudflare-astro
% npm run dev
```

![](https://storage.googleapis.com/zenn-user-upload/8252fd4fcf13-20230421.png)

## Astroプロジェクトを、IDEで開こう

VSCodeなどのIDEで、`ws-stripe-cloudflare-astro`ディレクトリを開きましょう。

![](https://storage.googleapis.com/zenn-user-upload/e473929e06ee-20230421.png)

### [Optional] VSCode拡張を追加する
Astroではページを`.astro`ファイルで保存します。

`.astro`拡張子をサポートしていないIDEでは、拡張機能を追加することをお勧めします。

https://marketplace.visualstudio.com/items?itemName=astro-build.astro-vscode


## おさらい

- AstroはUIライブラリ・FWに依存しないJSフレームワーク
- 対話形式でプロジェクトがセットアップできる
- `.astro`をサポートするIDE拡張機能があると便利

次のステップでは、セットアップしたアプリをCloudflare Pageにデプロイします。