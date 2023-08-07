---
title: "Introduction: Setting the Stage for the workshop"
---

早速ワークショップを始めましょう。

まずは必要なツールやアカウントを用意します。

## Stripeアカウントの作成

データを登録するためには、Stripeアカウントが必要です。
まだアカウントを持っていない方は、以下のURLからアカウントを作成してください。

https://dashboard.stripe.com/register

![](https://storage.googleapis.com/zenn-user-upload/939f923c3dfc-20220419.png)

アカウントを作成後、ダッシュボードにログインしましょう。

https://dashboard.stripe.com/
### 新しいテスト用アカウントを作成しよう

まずは、今回のテストアプリ用のストアアカウントを作成します。

Stripeでは、ストアやサービス毎に複数のアカウントを作成・管理することが可能です。
アカウントを分けることで、APIの動作確認やワークショップ中の事故で顧客データや請求内容に予期せぬ変更が入るリスクなどを回避できます。

ページ左上のアイコンをクリックし、[新規アカウント]をクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/3fcedd882d34-20220419.png)

アカウントでは、ビジネス・企業名とどの国で業務を行なっているかの２点を設定します。

今回はデモアプリですので、以下のように設定しましょう。

- アカウント名: demo.furni.com
- 業務をおこなっている国: Japan

![](https://storage.googleapis.com/zenn-user-upload/7f654a649b6b-20220419.png)

設定が完了すると、アカウントのTOPページに移動します。

![](https://storage.googleapis.com/zenn-user-upload/9058b6a5d7ce-20220419.png)

#### Tips: 本番環境での利用について

アカウント作成すると、テスト環境が用意されます。

テスト環境では、「Stripeが用意するテストカード番号以外の入力を拒否する」「請求書などのメールを送信できるアドレスが制限されている」などの制限があります。

実際に顧客の決済を受け付けるには、なるべく早い段階で本番環境の利用申請を行うようにしましょう。

![](https://storage.googleapis.com/zenn-user-upload/b0d0b1a434d3-20220419.png)

なお、「サービス内容の詳細などの情報が不十分だと判断された場合」や「[禁止/制限付き業種](https://stripe.com/ja-it/legal/restricted-businesses)に該当すると判断された場合」、アカウントが停止される可能性があります。禁止・制限付き業種に該当しないかの事前に確認や、できるだけ詳細なビジネス情報の入力をお勧めします。

## Stripe CLIのインストール

ワークショップ内で、StripeのWebhookイベントをローカル環境のアプリに転送する必要があります。

この転送にStripe CLIを利用しますので、インストールしましょう。

### Homebrewを使ったインストール
Homebrewを使ってのインストールが簡単です。

```
brew install stripe/stripe-cli/stripe
```

### macOS / Windowsでのインストール
macOSやWindowsの場合、GitHubのReleaseからダウンロードします。

https://github.com/stripe/stripe-cli/releases/latest

macOSの場合は「stripe_X.X.X_mac-os_x86_64.tar.gz」Windowsの場合は「stripe_X.X.X_windows_x86_64.zip」のように、各OS毎にDLファイルが用意されています。

これをDLし、実行することでStripe CLIを利用できるようになります。

その他の環境でのインストール方法は、Stripeドキュメントを確認してください。

https://stripe.com/docs/stripe-cli#install

:::message
**インストールできない場合**
もしStripe CLIのインストールがうまくいかない場合、ブラウザ上で利用できる[Stripe Shell]をご利用ください。

ダッシュボードにログインした状態で、[Stripe CLIのドキュメントページ](https://stripe.com/docs/stripe-cli)に移動しましょう。

[オンラインで試してみる]ボタンが表示されますので、クリックします。
![](https://storage.googleapis.com/zenn-user-upload/0344d992f943-20220420.png)


CLI操作画面がブラウザ下部に表示されます。
![](https://storage.googleapis.com/zenn-user-upload/b09650f24d2e-20220420.png)

:::


## Stripe CLIでStripeアカウントに接続する

Stripe CLIのインストールが終われば、次はStripeアカウントへの接続です。

Stripe CLIはStripe APIを介してStripeのリソースにアクセスします。そのため、Stripeアカウントに接続してAPIキーを取得する必要があります。


### stripe loginでStripeアカウントと接続
Stripeアカウントとの連携は、stripe loginコマンドで行います。
コマンドを実行すると、「Enterキーを押してブラウザを開いてね」とメッセージが表示されますので、[Enterキー]を押しましょう。

```
stripe login
Your pairing code is: amuse-revive-timely-sturdy
This pairing code verifies your authentication with Stripe.
Press Enter to open the browser (^C to quit)
```

:::message
**すでに別アカウントでログイン済みの場合**
Stripe CLIでは、`--project-name`を利用して複数のStripeアカウントに接続できます。
デフォルトの接続先アカウントを別のStripeアカウントにしたい場合は、これ以降登場する`stripe`コマンドを次のように変更しましょう。

```bash
stripe --project-name demo-furni
```

**例: 顧客一覧を取得する**

```bash
stripe --project-name demo-furni customers list
```

:::

Stripe Dashboardにログインしている状態であれば、以下の画像のようにアクセス権の確認画面が表示されます。

![](https://storage.googleapis.com/zenn-user-upload/d5ecd73d2edf-20230731.png)

[アクセスを許可]をクリックすると、完了画面が表示されます。

![](https://storage.googleapis.com/zenn-user-upload/8d56b179f6a7-20230731.png)

Stripe CLIの画面に戻ると、接続が成功したことが英文で表示されています。

```
> Done! The Stripe CLI is configured for api-test with account id acct_xxxxxxxx

Please note: this key will expire after 90 days, at which point you'll need to re-authenticate.
```

準備はできましたか？それでは早速ワークショップを始めていきましょう。

## 開発するe-commerceサイトの設定について

せっかくのワークショップですので、「なぜこのシステムを開発するのか」の背景を想像してみましょう。

あなたはとある家具販売会社のe-commerce開発プロジェクトに参加しています。

この会社では、高品質な家具を、ユーザーから注文されてから製造し、配達しています。

ウェブサイトへのアクセスはあまり多くないため、現時点では在庫の管理や注文数の制限を行う必要はありません。

商品の販売システムをあなたが準備すると、チームメンバーがサイトのデザインやコンテンツの追加を開始します。

そのため、できるだけ短いスケジュール・少ないコードで商品販売機能を作ることが求められています。

・・・イメージできましたでしょうか？

それでは、Stripeを使って商品販売機能を組み込む方法を体験していきましょう。