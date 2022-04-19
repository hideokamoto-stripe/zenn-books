---
title: "Stripeに登録した商品を、Next.jsで取得・表示しよう"
---

このステップでは、Stripeに登録した商品データをサイトに表示させる方法を体験します。

## StripeのAPIキーを取得しよう

Stripeのデータを取得するには、APIキーを利用します。
APIキーを確認するには、ダッシュボードの[開発者]メニューを開きましょう。

![](https://storage.googleapis.com/zenn-user-upload/95097c74935d-20220419.png)

続いて、左側のメニューから[APIキー]をクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/6fce96bb18c1-20220419.png)

### Tips: Stripe 3つのAPIキー

APIキーには3つのAPIキーが存在します。

1つ目は「公開可能キー」です。これはフロントエンドアプリで、カードのtoken化などに利用します。
2つ目は「シークレットキー」です。こちらはStripeの公開しているAPI全てにアクセスができるAPIで、万が一漏洩した場合には顧客情報や注文データなどを取得されるリスクが存在します。
最後のキーは、「制限付きのキー」です。シークレットキー同様にStripeのAPIにアクセスするために利用しますが、アクセスできるリソースや操作を個別に設定することができます。

公開可能キーとシークレットキーの２つがあれば、Stripeを利用したアプリの開発が可能です。
ただし、より安全にStripeアカウントを運用したい場合は、シークレットキーの代わりに制限付きのキーを利用することをお勧めします。

### 制限付きのキーーを作成しよう

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

## Next.jsの環境変数に、APIキーを登録しよう

取得したAPIキーは、Next.jsの環境変数からアプリで利用します。

`.env.local`ファイルを作成し、以下のようにAPIキーを設定しましょう。

**.env.local**


```
STRIPE_API_KEY=rk_test_xxxxx
NEXT_PUBLIC_STRIPE_PUBLISHABLE_API_KEY=pk_test_xxxxxx
```

- `STRIPE_API_KEY`は先ほど作成した制限付きのキーを入力します。
- `NEXT_PUBLIC_STRIPE_PUBLISHABLE_API_KEY`には、公開可能キーを入力しましょう。

公開可能キーは、Reactアプリ側で利用します。そのため、`NEXT_PUBLIC_`をつけることでフロントエンドでも利用できるように設定しています。

制限付きのキーなど、第三者に見せたくない値については`NEXT_PUBLIC_`をつけないように注意しましょう。

## 商品一覧を返すREST APIを実装しよう

## 商品一覧ページを実装しよう

### `pages/products.jsx`を作成しよう

### `getStaticProps`で商品データを取得しよう

#### `revalidate`を設定して、キャッシュの自動更新を行おう

## おさらい

- Stripeのデータを取得するには、シークレットAPIキーまたは制限付きAPIキーを利用する
- `getStaticProps`でAPIのレスポンスを利用したページを事前にビルドできる
- `revalidate`を設定すると、一定間隔でページの再ビルドを行える

次のステップでは、商品の注文ボタンを設定します。

## 「早く終わった」という方のための、もう１ステップ

C2Cのマーケットプレイスを構築するケースやタイムセールを行いたい場合などでは、「常に最新の商品一覧を表示する」必要があります。

useSWRライブラリを組み合わせることで、最新の情報をAPIから取得し表示させることができます。

以下のドキュメントを参考に、商品一覧ページを変更してみましょう。

https://swr.vercel.app/ja/docs/with-nextjs

https://swr.vercel.app/ja/docs/revalidation