---
title: "Chapter1: Amazon EventBridgeクイックスタートで、AWSでStripeのイベントを受信しよう"
---

まずは、AWSのクイックスタートを利用して、Stripe内のイベントをAWSで処理するためのイベントバスを作成しましょう。

## Step0: AWS Lambdaの制限状況を確認する

事前準備として、AWS Lambdaをセットアップできる状況かを確認しましょう。

マネージメントコンソール上部の検索フォームで、[lambda]と入力します。

![](https://storage.googleapis.com/zenn-user-upload/6a5ae17becf8-20221015.png)

[Lambda]を選択すると、AWS Lambdaの管理画面が表示されます。

![](https://storage.googleapis.com/zenn-user-upload/2f3f42d4e2c6-20221015.png)

ここで、[予約されていないアカウントの同時実行]の数字を確認してください。

![](https://storage.googleapis.com/zenn-user-upload/4fdae2e96b17-20221015.png)

数字が10またはそれ以下の場合は、[同時実行数の上限]を更新する必要があります。

変更方法については、以下のドキュメントまたは、ワークショップのアシスタントに質問して確認してください。

https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/provisioned-concurrency.html#configuring-provisioned-concurrency

## Step1: Amazon EventBridgeクイックスタートを開始する

確認と設定の変更が終われば、早速クイックスタートでのセットアップを開始しましょう。

AWSマネジメントコンソール上部の検索フォームで、[eventbridge]と入力します。

![](https://storage.googleapis.com/zenn-user-upload/318a4a4c9ad4-20221015.png)

EventBrdigeをクリックすると、管理画面のトップページに移動します。

![](https://storage.googleapis.com/zenn-user-upload/a3c3d3f11eb2-20221015.png)

左側のメニューから、[クイックスタート]を選択すると、２つの選択肢が表示されます。

![](https://storage.googleapis.com/zenn-user-upload/a736bd13b7ba-20221015.png)

StripeまたはTwilio / GitHubと連携させたい場合、下部の[Lambda関数URLを利用したインバウンドウェブフック]を利用します。

[使用を開始する]リンクをクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/382c50d201f9-20221015.png)

連携するサービスに応じたテンプレート選択画面が表示されます。

![](https://storage.googleapis.com/zenn-user-upload/2a01d7f074f8-20221015.png)

今回はStripeですので、[Stripe]の[設定]をクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/64a59d160d24-20221015.png)

StripeでのインバウンドWebhookをセットアップするためのページが表示されました。

![](https://storage.googleapis.com/zenn-user-upload/3d4ae895b5d5-20221015.png)
## Step2: クイックスタートから、CloudFormationを起動しよう

Stripe・GitHub・TwilioとAWSを連携するためのリソースは、CloudFormationを利用して立ち上がります。

そのため、ここからはCloudFormationを利用したセットアップ方法を紹介します。

まず[ステップ１]でイベントバスを選択します。これは初期状態の`default`のまま進めましょう。

![](https://storage.googleapis.com/zenn-user-upload/ea7caca4e652-20221015.png)

続いて[ステップ2]でCloudFormationの起動フローに移動します。

[新しいStripeウェブフック]ボタンをクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/126c5b345f89-20221015.png)

「Lambdaの関数URLが一般にアクセスできる形で生成される」ことに対する確認画面が表示されます。

今回立ち上げるLambdaのコードには、Stripeからのリクエストか否かを検証する処理が含まれていますので、チェックボックスをオンにして[確認]ボタンをクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/efecfa4203e0-20221015.png)

CloudFormation立ち上げ画面が開きます。

「すでにクイックスタートでスタックを立ち上げたことのある人」以外は、スタックの名前をデフォルトのまま進めましょう。

作成したことのある方は、一度過去に作成したスタックを削除するか、この作成ステップをスキップしてください。

![](https://storage.googleapis.com/zenn-user-upload/220c32be54c1-20221015.png)

CloudFormationの起動に必要なパラメータを指定します。[StripeWebhookSecret]が空欄ですので、一旦[test]と入力しましょう。

この値は後ほど変更します。

![](https://storage.googleapis.com/zenn-user-upload/4be3992a5c71-20221015.png)

[機能と変換]でチェックボックスが表示されます。これはCloudFormationがIAMリソースを作成するなどの兼ね合いがあるため表示されています。

すべてにチェックをつけて、[スタックの作成]をクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/455ad22927f2-20221015.png)

作成画面に移動します。

![](https://storage.googleapis.com/zenn-user-upload/c3cecae5879a-20221015.png)

ステータスが[CREATE_COMPLETE]に変われば作成完了です。

![](https://storage.googleapis.com/zenn-user-upload/2f6b5c82fb24-20221015.png)

## Step3: Stripe Webhookに、Lambda Functions URLを登録しよう

StripeのWebhookイベントを受け付けるためのURLが生成できました。

ここからは、StripeのWebhookへの紐付けと、署名検証の設定を行います。

先ほど作成したCloudFormationの、[出力]タブをクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/393d82abbe9c-20221015.png)

[FunctionURLEndpoint]にURLが表示されていますので、コピーします。

![](https://storage.googleapis.com/zenn-user-upload/faf340ce49b0-20221015.png)


### 3-1: Stripeダッシュボードにログインしよう

まずはStripeダッシュボードにログインしましょう。

![](https://storage.googleapis.com/zenn-user-upload/66f726dd3f1d-20221005.png)

アカウントを作成していない場合は、https://dashboard.stripe.com からアカウントを新規に作成できます。

本番利用の申請を行わない場合、メールアドレスと氏名だけでアカウントを作ることができます。

![](https://storage.googleapis.com/zenn-user-upload/c7759598b3c3-20221005.png)

#### すでに本番利用しているアカウントをお持ちの場合

本番利用中のStripeアカウントをお持ちの場合、アカウントを新しく追加することをお勧めします。

アカウントを分けることで、「URLやAPIキーを本番環境のものと取り違えるリスクを減らすこと」や「システム動作確認のノイズになるテストデータの混入防止」などが期待できます。

Stripeでは、ダッシュボードログイン後、左上のアカウント名をクリックすることで、[新規アカウント]ボタンをクリックできます。

![](https://storage.googleapis.com/zenn-user-upload/7755ffde8c70-20221005.png)

クリックすると、アカウント名とビジネスを提供する国を指定する画面が立ち上がります。

名前は`Amazon EventBridgeワークショップ`を、国は`日本`を選択しましょう。

![](https://storage.googleapis.com/zenn-user-upload/88c59395e810-20221005.png)

### 3-2: Lambda Functions URLをStripe Webhookに登録しよう

[開発者]タブをクリックし、左側のメニューから[Webhook]を選択しましょう。

![](https://storage.googleapis.com/zenn-user-upload/f584383f5c0a-20221005.png)

Webhookを作成していない場合は、画面の中央下部に[エンドポイントを追加]ボタンが表示されます。

すでにWebhookを作成しているアカウントでは、画面右上部に[エンドポイントを追加]ボタンが表示されてます。

どちらかのボタンをクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/b92b94ca31cc-20221015.png)

Webhookの設定画面が開きます。

右側にサンプルコードが表示されますので、コードを書いて組み込みする場合は、これをベースに開発しましょう。

今回は、AWSに処理を委ねるため、右側のコードは無視します。

![](https://storage.googleapis.com/zenn-user-upload/86896e667204-20221015.png)

[エンドポイントURL]に、先ほどCloudFormationの[出力]タブでコピーしたURLを貼り付けます。

![](https://storage.googleapis.com/zenn-user-upload/867e5e317f7f-20221015.png)

[説明]と[リッスンする]そして[バージョン]は、デフォルトのまま進めましょう。

![](https://storage.googleapis.com/zenn-user-upload/2b818252f9ed-20221015.png)

[リッスンするイベント]にて、[イベントを選択]をクリックします。

![](https://storage.googleapis.com/zenn-user-upload/9a415b51a15d-20221015.png)

Stripeから送信できるイベントのリストが表示されます。

検索フォームに[customer.subscription]と入力しましょう。

![](https://storage.googleapis.com/zenn-user-upload/cd9de095e685-20221015.png)

`customer.subscription.created`と`customer.subscription.deleted`の２つにチェックを入れます。

![](https://storage.googleapis.com/zenn-user-upload/714903837b2e-20221015.png)

[イベントを追加]ボタンをクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/629d44d1099d-20221015.png)

選択した２つのイベントが追加されました。

[イベントを追加]ボタンをクリックして、登録を完了させます。

![](https://storage.googleapis.com/zenn-user-upload/c88bc2e7d5dc-20221015.png)

下の画像と同じ、Webhookの設定詳細画面が表示されれば、OKです。

![](https://storage.googleapis.com/zenn-user-upload/62f77d4bb278-20221015.png)

### 3-3: Webhookの署名シークレットを取得し、AWSに設定しよう

Webhook APIに対して、Stripe以外からリクエストが送信できてしまうと、不正なデータが紛れ込むリスクが発生します。

そのため、署名検証のためのシークレットキーを取得して、AWSのクイックスタートで起動したLambdaにて検証処理を実施しましょう。

![](https://storage.googleapis.com/zenn-user-upload/62f77d4bb278-20221015.png)

先ほどの画面に、[署名シークレット]項目が表示されています。

![](https://storage.googleapis.com/zenn-user-upload/f1969945f088-20221015.png)

[表示]をクリックすると、`whsec_xxx`から始まるキーが表示されます。このキーをコピーしましょう。

![](https://storage.googleapis.com/zenn-user-upload/aaec1a308503-20221015.png)

AWSのマネジメントコンソールに戻り、CloudFormation管理画面で、先ほど立ち上げたクイックスタートリソースを選びます。

![](https://storage.googleapis.com/zenn-user-upload/64d1138c84c1-20221015.png)

スタック名（`StripeInboundWebhookStack`）の右に、[更新]ボタンがあるのでクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/64d1138c84c1-20221015.png)

CloudFomartionテンプレートそのものを変更するかを聞かれます。

今回は変更しませんので、[現在のテンプレートを使用]を選んで、[次へ]をクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/a67632c53b02-20221015.png)

続いてパラメータ設定画面が開きます。

![](https://storage.googleapis.com/zenn-user-upload/feefdd967ff2-20221015.png)

`StripeWebhookSecret`がロックされていますので、[現在の値を使用]のチェックボックスを外しましょう。

![](https://storage.googleapis.com/zenn-user-upload/b1906116ff52-20221015.png)

先ほどStripeダッシュボードでコピーした、`whsec_`から始まるキーを、入力します。

![](https://storage.googleapis.com/zenn-user-upload/3b626644e13b-20221015.png)

[次へ]をクリックすると、タグなどのカスタマイズ画面が開きます。

ここの変更は行いませんので、スクロールして[次へ]を選びましょう。

![](https://storage.googleapis.com/zenn-user-upload/f2a6a2a139bf-20221015.png)

レビュー（確認）画面が開きます。

![](https://storage.googleapis.com/zenn-user-upload/d36eb51a6a3e-20221015.png)

スクロールすると、変更するリソースの表示や、確認のチェックボックスが表示されます。

チェックボックスをオンにして、[スタックの更新]をクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/f43f1d4ad18c-20221015.png)

これでスタックの更新が完了すれば、署名検証の準備まで完了です。

## おさらい

- Amazon EventBridgeでは、クイックスタートとして特定のSaaSとの連携準備が簡単にできる
- Stripe/GitHub/Twilioの場合、Lambda Functions URLを利用するため、署名検証処理で不正なリクエストを予防する必要がある
- 先にリソースを作成して、URLを取得してから、署名シークレットをStripeダッシュボードで発行する

次のステップでは、一旦EventBridgeから離れて、「WordPressを立ち上げるワークフローのデモ」をStep Functionsで起動させます。