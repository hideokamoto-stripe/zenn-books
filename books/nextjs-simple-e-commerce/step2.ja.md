---
title: "Step 1: ローコードでシンプルな製品販売ページを実装する"
---

このステップでは、できるだけ少ないコードで、シンプルな製品販売ページを追加する方法を紹介します。

## Next.js(version 13)でウェブアプリをセットアップしよう

まずはウェブアプリを用意しましょう。

この資料では、Next.js(version 13)を利用します。

`create-next-app`コマンドを実行して、アプリをセットアップしましょう。


https://www.npmjs.com/package/create-next-app

```bash
npx create-next-app@13
```
:::message
Next.jsドキュメントでは、`$ npx create-next-app@latest`を推奨しています。
ですが、このワークショップ資料では、Next.js version 13系列（資料作成時はversion `13.4.12`を利用）を前提として作成しています。

そのため、資料では便宜上、必ずVersion 13を利用するように設定しています。
:::

「`create-next-app`コマンドをダウンロードしてよいか？」を確認されますので、[y]を入力しましょう。

```bash
Need to install the following packages:
  create-next-app@13.4.12
Ok to proceed? (y) 
```

ダウンロードが完了すると、対話形式でプロジェクトのセットアップが始まります。

まずはプロジェクト名を聞かれますので、`nextjs-stripe-e-commerce`と入力しましょう。

```bash
? What is your project named? › nextjs-stripe-e-commerce
```

「TypeScriptを利用するか」を聞かれますので、これも`y`と回答します。

```bash
? Would you like to use TypeScript? › No / Yes
```

「ESLintを利用するか」を聞かれますので、これも`y`と回答します。


```bash
? Would you like to use ESLint? › No / Yes
```

次の質問に`y`と回答すると、Tailwind CSSがセットアップされます。


```bash
? Would you like to use Tailwind CSS? › No / Yes
```

ディレクトリ構造について質問があります。`src`ディレクトリは利用しないので、`no`と回答しましょう。


```bash
? Would you like to use `src/` directory? › No / Yes
```

[App Router]はNext.js version 13から追加された機能です。

このワークショップで利用しますので、`yes`を選びます。


```bash
? Would you like to use App Router? (recommended) › No / Yes
```

最後に[Importエイリアス]について聞かれます。こちらもデフォルトの`no`を選びましょう。


```bash
? Would you like to customize the default import alias? › No / Yes
```

ここまでの設定が終われば、アプリのセットアップが始まります。

```bash

Creating a new Next.js app in /Users/hideokamoto/stripe/sandbox/javascript/nextjs-stripe-e-commerce.

Using npm.

Initializing project with template: app-tw 


Installing dependencies:
- react
- react-dom
- next
- typescript
- @types/react
- @types/node
- @types/react-dom
- tailwindcss
- postcss
- autoprefixer
- eslint
- eslint-config-next
```

次のようなメッセージが表示されれば、セットアップ完了です。


```bash

131 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
Initialized a git repository.

Success! Created nextjs-stripe-e-commerce at /Users/stripe/sandbox/nextjs-stripe-e-commerce
```

## Next.jsアプリをローカル環境で起動する

セットアップしたコードを、ローカルで動かしてみましょう。

```bash
npm run dev
```

このコマンドを実行すると、作業中のPC内でアプリが立ち上がります。

```bash
$ next dev
- ready started server on 0.0.0.0:3000, url: http://localhost:3000
```

もし`3000`ポートが他のアプリで使われている場合は、空いているポートを探して利用します。

```bash
$ next dev
- warn Port 3000 is in use, trying 3001 instead.
- ready started server on 0.0.0.0:3001, url: http://localhost:3001
```

このURLにブラウザからアクセスすると、セットアップしたNext.jsアプリのデフォルトページが表示されます。

![](https://storage.googleapis.com/zenn-user-upload/24d5928f6859-20230728.png)

## Next.js v13でレイアウトを設定する

Next.jsでは、`app/layout.tsx`を利用してサイト全体のレイアウトを設定できます。

https://nextjs.org/docs/app/building-your-application/routing/pages-and-layouts

早速、ヘッダーを追加してみましょう。

`app/layout.tsx`を次のように変更してください。

```diff:typescript app/layout.tsx

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body className={inter.className}>
+        <header className="relative bg-white">
+          <nav aria-label="Top" className="max-w-7xl mx-auto py-2 px-4 sm:px-6 md:px-8">
+            <div className="h-16 flex items-center">
+              <div className="hidden md:ml-8 md:block md:self-stretch">
+                <div className="h-full flex space-x-8">
+                  <a href="/" className="flex items-center text-sm font-medium text-gray-700 hover:underline">Home</a>
+                  <a href="/products" className="flex items-center text-sm font-medium text-gray-700 hover:underline">Products</a>
+                </div>
+              </div>
+              <div className="ml-auto flex items-center">
+                <div className="hidden md:flex md:flex-1 md:items-center md:justify-end md:space-x-6">
+                  <a href="/" className="flex items-center text-sm font-medium text-gray-700 hover:underline">My page</a>
+                </div>
+              </div>
+            </div>
+          </nav>
+        </header>
        {children}
      </body>
    </html>
  )
}
```

変更を保存すると、自動的にページが再読み込みされます。

そしてヘッダーメニューが追加されていれば、編集成功です。

![](https://storage.googleapis.com/zenn-user-upload/ddf58cb9e85d-20230728.png)

## 販売する商品をStripeに登録しよう

ウェブアプリの準備ができましたので、掲載する商品をStripeに登録しましょう。

ブラウザで、Stripeダッシュボードを開きます。

ダッシュボードのメニューから[商品]をクリックします。

![](https://storage.googleapis.com/zenn-user-upload/6deb907038be-20230804.png)
直接リンク: https://dashboard.stripe.com/test/products

[商品を追加]ボタンがありますので、クリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/d4eb3a31930f-20230804.png)


商品の登録画面が開きます。

### 商品の基本情報を登録しよう

デモ用の商品として、以下のテキストと画像を入力・アップロードしましょう。

|商品名|画像（フリー素材DLページ）|
|:--|:--|
|ソファ|https://images.unsplash.com/photo-1616627451515-cbc80e5ece35?ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&ixlib=rb-1.2.1&fm=png&fit=crop&w=600&q=80|

![](https://storage.googleapis.com/zenn-user-upload/1f65c05d8c8d-20230804.png)

### 価格情報（料金）を登録して保存しよう

続いて商品の価格を登録しましょう。

ページをスクロールして[料金情報]フォームを表示し、以下のデータを入力します。

|料金体系モデル|価格|通貨|種類|
|:--|:--|:--|:--|
|標準の料金体系|150000|JPY|[一括]|

![](https://storage.googleapis.com/zenn-user-upload/49502b8f841d-20230804.png)

入力が終われば、ページ右上の[商品を保存]をクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/bd43e47770c1-20230804.png)


商品ページに移動すれば、登録完了です。

![](https://storage.googleapis.com/zenn-user-upload/31df2dfcd66a-20230804.png)


:::message
### 複数の商品を追加しよう

テスト用のデータとして、いくつかのサンプルを以下に用意しました。
全て登録する必要はありませんが、多い方が通販サイトらしくなるかと思いますので、時間に余裕のある方はぜひ登録してみましょう。

|商品名|価格|画像（フリー素材DLページ）|料金体系モデル|通貨|種類|
|:--|:--|:--|:--|:--|:--|
|Chair|97000|[Link](https://images.unsplash.com/photo-1622147681210-d7da05b4a7d7?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&fm=png&fit=crop&w=600&q=80)|標準の料金体系|JPY|[一括]|
|Wooden Stool|6000|[Link](https://images.unsplash.com/photo-1503602642458-232111445657?ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&ixlib=rb-1.2.1&fm=png&fit=crop&w=600&q=80)|標準の料金体系|JPY|[一括]|
|Retro Chair|12000|[Link](https://images.unsplash.com/photo-1519947486511-46149fa0a254?ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&ixlib=rb-1.2.1&fm=png&fit=crop&w=600&q=80)|標準の料金体系|JPY|[一括]|
:::

## 商品注文リンクとボタンを作成しよう

Stripeでは、注文ページをノーコードで作成することができます。

注文ページのURLをダッシュボードから発行してみましょう。


ダッシュボードを表示した状態で、 [c]キー -> [l]キーの順にキーボードを入力します。

![](https://storage.googleapis.com/zenn-user-upload/9058b6a5d7ce-20220419.png)

すると支払いリンクを作成する画面が開きます。

![](https://storage.googleapis.com/zenn-user-upload/6077c0b5f263-20230525.png)

このように、Stripeダッシュボードでは、操作を簡単にするための[キーボードショートカット]が複数用意されています。

![](https://storage.googleapis.com/zenn-user-upload/85f46fa1c303-20230526.png)

:::message
#### その他の支払いリンク作成画面の開き方
キーボードショートカット以外にも、いくつかのアクセス方法が用意されています。

##### ヘッダーメニューから移動する
ページ上部の[作成]ボタンをクリックし、[支払いリンク]を選択しましょう。
![](https://storage.googleapis.com/zenn-user-upload/69bed5e60a70-20230525.png)

##### URLで直接移動する
https://paymentlinks.new をブックマークすることで、URLから直接移動することも可能です。
:::

Payment Links作成ページでは、注文できる商品を登録します。

![](https://storage.googleapis.com/zenn-user-upload/6077c0b5f263-20230525.png)


画面左側の設定画面にある、[商品]セクションの検索フォームをクリックしましょう。

[新しい商品を追加] ボタンと、すでに登録されている商品のリストが表示されます。

![](https://storage.googleapis.com/zenn-user-upload/e1637ddc7ebb-20230525.png)

[Sofa]商品を選びましょう。


これで編集中の支払いリンクに 商品が追加されました。
![](https://storage.googleapis.com/zenn-user-upload/d14f28553922-20230525.png)

あとは画面右上にある[リンクを作成]を クリックすると・・・
![](https://storage.googleapis.com/zenn-user-upload/f9a16f0361fa-20230525.png)

これだけで注文用のURLが発行できました。

![](https://storage.googleapis.com/zenn-user-upload/e45aca6b0e93-20230525.png)

表示されているURLをコピーしたり、QRコードをダウンロードして共有することで、顧客からの注文・決済を受け付けることができます。

### 購入ボタンのコードスニペットを埋め込む

2023年から、支払いリンクをボタンまたはカード形式でサイトに埋め込むことができるようになりました。

https://stripe.com/docs/payment-links/buy-button


支払いリンク管理画面にある、[購入ボタン]ボタンをクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/7f4ba71e8e68-20230525.png)

クリックすると、サイトやアプリに埋め込む購入ボタンの編集画面が開きます。

![](https://storage.googleapis.com/zenn-user-upload/a7a7926ab354-20230526.png)

[コードをコピー]をクリックしましょう。

コピーしたコードを貼り付けるだけで、ウェブサイトやアプリに注文・サブスクリプション申し込み機能を追加できます。

## Next.js v13で商品ページを作成する

コピーしたコードスニペットを利用して、ウェブアプリに商品ページを追加しましょう。

### App Routerで商品ページを追加する

まずは商品ページ用のファイルを作成しましょう。

`app/products/page.tsx`を作成します。

商品ページのレイアウトを作りましょう。

```ts:app/products/page.tsx
import { Metadata } from 'next'
 
export const metadata: Metadata = {
  title: 'Products',
}
 
export default function Page() {
  return (
    <main className="flex min-h-screen flex-col items-center justify-between p-24">
        <div className="mb-32 grid text-center lg:mb-0 lg:grid-cols-4 lg:text-left md:gap-10">
            <h1 className='text-4xl font-extrabold'>Products</h1>

         </div>
    </main>
  )
}

```

![](https://storage.googleapis.com/zenn-user-upload/54aa6315092e-20230804.png)

コピーしたコードを、貼り付けます。

```diff ts:app/products/page.tsx
export default function Page() {
  return (
    <main className="flex min-h-screen flex-col items-center justify-between p-24">
        <div className="mb-32 grid text-center lg:mb-0 lg:grid-cols-4 lg:text-left md:gap-10">
            <h1 className='text-4xl font-extrabold'>Products</h1>
+            <script async
+                src="https://js.stripe.com/v3/buy-button.js">
+            </script>

+            <stripe-buy-button
+                buy-button-id="buy_btn_xxxx"
+                publishable-key="pk_test_xxxxxx"
+            >    
+            </stripe-buy-button>
        </div>
    </main>
  )
}

```

これでStripeに登録した商品を表示できました。

![](https://storage.googleapis.com/zenn-user-upload/eadac29a7f35-20230804.png)

:::message
***[Advanced] 複数の商品を登録しよう***

前のステップで、複数の商品を登録された方は、このステップを繰り返して商品一覧を作ってみましょう。

Buy Buttonを複数追加する場合は、`script`タグは1ページに1つだけでOKです。


```diff:tsx app/products/page.tsx
export default function Page() {
  return (
    <main className="flex min-h-screen flex-col items-center justify-between p-24">
        <div className="mb-32 grid text-center lg:mb-0 lg:grid-cols-4 lg:text-left">
            <h1 className='text-4xl font-extrabold'>Products</h1>
            <script async
                src="https://js.stripe.com/v3/buy-button.js">
            </script>
            <stripe-buy-button
                buy-button-id="buy_btn_xxxx"
                publishable-key="pk_test_xxxxxx"
            >    
            </stripe-buy-button>

+            <stripe-buy-button
+                buy-button-id="buy_btn_xxxx"
+                publishable-key="pk_test_xxxxxx"
+            >    
+            </stripe-buy-button>
        </div>
    </main>
  )
}
```

![](https://storage.googleapis.com/zenn-user-upload/174da718a960-20230804.png)

:::

### TypeScriptのエラーを解消する

TypeScriptで開発している場合、次の型に関するエラーが発生します。

```
./app/products/page.tsx:17:9
Type error: Property 'stripe-buy-button' does not exist on type 'JSX.IntrinsicElements'.
```

このエラーを解消するには、`types/stripe.d.ts`ファイルを追加しましょう。

```ts:types/stripe.d.ts
declare namespace JSX {
    interface IntrinsicElements {
      'stripe-buy-button': {
        'buy-button-id': string;
        'publishable-key': string;
        children: unknown;
      };
    }
  }
```

これでエラーがなくなります。


## Stripeのテストカード情報で、実際に注文してみよう

商品ページが作成できたので、実際に注文をしてみましょう。

Stripeでは、開発中の動作確認のため、テスト環境限定の決済情報を複数用意しています。

https://stripe.com/docs/testing

### クレジットカード決済で注文してみよう

商品ページで[注文]ボタンをクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/eadac29a7f35-20230804.png)

Stripeのリダイレクト型決済フォームが表示されます。

![](https://storage.googleapis.com/zenn-user-upload/f58d3f162732-20230804.png)

ここで、以下のカード情報を入力しましょう。

|Field|Value|
|:--|:--|
|Card number| `4242424242424242`|
|Card expiration| `12`/`30` |
|CVC| `424`|
|Name| `Demo customer`|
|Email| `test@example.com`|

![](https://storage.googleapis.com/zenn-user-upload/c4cc8b50de79-20230804.png)

注文ボタンをクリックすると、クレジットカードでテスト支払いが行われます。

![](https://storage.googleapis.com/zenn-user-upload/879e2e600bb7-20230804.png)

Stripeダッシュボードの[支払い]タブをみると、`Demo customer`からの注文が表示されます。

![](https://storage.googleapis.com/zenn-user-upload/f84077cacd4a-20230804.png)

:::message
**[Advanced] 決済失敗や不正利用をシミュレーションしよう**

Stripeでは、「クレジットカードの利用残高が不足している場合」や「商品が届かないと、クレジットカード会社にクレームが届いた場合」などのテストもできます。

![](https://storage.googleapis.com/zenn-user-upload/ed8b802b5d00-20230804.png)

ドキュメントサイトから、３パターンのテストケースをピックアップしましたので、ぜひお試しください。

|Field|Value|
|:--|:--|
|Card number (カードの残高不足)| `4000000000009995`|
|Card number (3Dセキュア認証)| `4000002760003184`|
|Card number (不審請求の申請)| `4000000000002685`|
|Card expiration| `12`/`30` |
|CVC| `424`|
|Name| `Demo customer`|
|Email| `test@example.com`|

:::

## おさらい

- サイト全体のレイアウトは、`app/layout.tsx`で設定する
- ページを追加する際は、`app/[パス名]/page.tsx`
- Stripeに商品を登録することで、決済フォームURLをノーコードで作れる
- Stripe Payment Linksは、埋め込み用のコードスニペットが生成できる
