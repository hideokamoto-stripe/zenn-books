---
title: "おわりに:Stripeのノーコードサービス + AWSで広がる「決済OPS」"
---

短時間のワークショップでしたが、無事完了できましたでしょうか？

今回はサーバーの立ち上げのみにフォーカスしましたが、StripeのCustomer Portalを利用すると、「解約時のサーバー停止・削除ワークフロー」も自動化できます。

![](https://storage.googleapis.com/zenn-user-upload/7fc3096f741a-20221006.png)

この場合、「Stripeのサブスクリプションメタデータ」や「Amazon DynamoDBなどのDBサービス」を利用して、StripeのサブスクリプションとAWSで立ち上げたCloudFormationスタックを紐づけるタスクが必要です。

Step Functionsを利用している場合は、DynamoDBのアクションを利用することで、ノーコードで実装が可能ですので、ぜひお試しください。

## 後片付け

### CloudFormationスタックの削除

**Step Functionsが起動するCloudFormationに、EC2インスタンスが含まれます**。

CloudFormation管理画面から、必ず起動したスタックを**全て削除**しましょう。

![](https://storage.googleapis.com/zenn-user-upload/f6c5aed82cd2-20221006.png)

また、Amazon EventBridgeのクイックスタートで起動したCloudFormationスタック`StripeInboundWebhookStack`についても、利用予定がない場合は削除をお勧めします。

![](https://storage.googleapis.com/zenn-user-upload/7b098d8b4856-20221006.png)

### S3バケットの削除

QuickSightなどでの分析のために、S3バケットを作成しています。

こちらも、S3管理画面から、「バケットを空にする」でファイルを削除しましょう。

![](https://storage.googleapis.com/zenn-user-upload/ddb040bf9637-20221006.png)

バケットそのものは「バケットを削除」から削除できます。

![](https://storage.googleapis.com/zenn-user-upload/95c93a2675de-20221006.png)

### CloudWatch Logsのロググループ削除

Step Functionsの実行ログがCloudWatch Logsに保存されています。

デフォルトでは有効期限が設定されていませんので、手動で削除しましょう。

Step Functions管理画面の[ステートマシン]一覧で、削除予定のステートマシンの[ログ]リンクをクリックしましょう。


![](https://storage.googleapis.com/zenn-user-upload/e927f17422a8-20221006.png)

[アクション]から[ロググループの削除]を選択すると、削除できます。

![](https://storage.googleapis.com/zenn-user-upload/15f475fc3f07-20221006.png)

### Step Functionsの削除

Step Functionsのステートマシンは、Step Functions管理画面の[ステートマシン]一覧から削除できます。

![](https://storage.googleapis.com/zenn-user-upload/9dc500eee740-20221006.png)

## ワークショップ資料について

本資料作成時点(2022/10)では、サーバー立ち上げのみにフォーカスした内容のみです。

が、今後「ノーコードでStripeとAWSを利用したサービス開発」をテーマにトピックを拡充予定です。

アップデートがあり次第、新しくワークショップ開催のお知らせなどを行いますので、ご期待下さい。