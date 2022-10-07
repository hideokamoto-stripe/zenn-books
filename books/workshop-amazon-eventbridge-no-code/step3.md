---
title: "Stripeのデータを、分析用にS3へ保存しよう"
---

SaaSの運用や改善・アップデートには、サブスクリプションの契約情報などのデータ分析が必要になります。

ここでは、サブスクリプション申し込み時のデータをS3に保存し、QuickSightやAthenaで分析する準備を行います。

## Step1: 分析用データを保存するS3バケットを作成しよう

まずはStripeのデータを保存するための、S3バケットを用意しましょう。

マネージメントコンソール上部の検索フォームに[s3]と入力しましょう。

検索結果、[S3]がでてきますので、クリックします。

![](https://storage.googleapis.com/zenn-user-upload/c9106d85a180-20221007.png)

続いて、バケットを作成します。

[バケット]の右側にある[バケットを作成]をクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/c7640c60444f-20221007.png)

今回は、デフォルト設定のままバケットを作成します。

![](https://storage.googleapis.com/zenn-user-upload/b7aae584ed78-20221007.png)

[バケット名]に[{英語で名前}-stripe-workshop-{今日の日付}]と入力しましょう。（例: `hidetaka-stripe-workshop-20221014`）

S3のバケット名は、ドメインなどと同様重複することができませんので、ご注意ください。

![](https://storage.googleapis.com/zenn-user-upload/f374ac065f07-20221007.png)

ページをスクロールして、[バケットを作成]ボタンをクリックします。

下の画像のように、[作成に成功した]メッセージが表示されれば完了です。

![](https://storage.googleapis.com/zenn-user-upload/dc94eb7015ea-20221007.png)
## Step2: Step　Functionsのステートマシンを編集し、S3にデータを保存しよう

続いて、作成したS3バケットにデータを投入するフローを追加します。

### 2-1: Step FunctionsのWorkflow Studioを開く

Step1で作成した、Step Functionsのステートマシンを開きましょう。

![](https://storage.googleapis.com/zenn-user-upload/b81d681e7aa7-20221007.png)

右側にある[編集]ボタンをクリックし、編集画面を開きます。

[Workflow Studio]ボタンが、ワークフローを図示している部分にありますので、クリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/4eac31375004-20221007.png)

クリックすると、GUIでワークフローを編集できる画面が立ち上がります。

### 2-2: S3にオブジェクトを追加するステップを追加

エディタを開けたので、新しいタスク（S3にデータを保存）を登録しましょう。

左側にある検索フォームに、[putobject]と入力します。

![](https://storage.googleapis.com/zenn-user-upload/0756a8043fff-20221007.png)


[PutObject]のカードをドラッグ&ドロップで、[CreateStack]の前におきましょう。

![](https://storage.googleapis.com/zenn-user-upload/f7ca672763e2-20221007.png)

これでワークフローの追加が完了しました。

### 2-3: 入力データと出力データを編集しよう

S3に出力するデータの内容やファイル名の設定を行いましょう。

ワークフロー内の[PutObject]をクリックすると、右側に設定画面が表示されます。

![](https://storage.googleapis.com/zenn-user-upload/f0829ce268a2-20221007.png)

[設定]タブの、[API]パラメータに以下のJSONを入力しましょう。

```json
{
        "Body.$": "$.detail.data.object",
        "ContentType": "application/json",
        "Bucket": "{先ほど作成したS3バケットの名前}",
        "Key.$": "$.id"
}
```

![](https://storage.googleapis.com/zenn-user-upload/3b98578c6209-20221007.png)

続いて、[出力]タブをクリックします。

![](https://storage.googleapis.com/zenn-user-upload/4837d9d63aac-20221007.png)

ここでは、次のステップに送るデータ内容を設定できます。

今回は、「PutObjectの結果」ではなく、「Start時の値」が次のステップに必要です。

そのため、[ResultPathを使用して元の入力を出力に追加]をオンにしましょう。

![](https://storage.googleapis.com/zenn-user-upload/6bf2fa3a48ea-20221007.png)

どのようにデータを追加するかの設定画面が表示されます。

[Discard result and keep original input]を選択しましょう。

![](https://storage.googleapis.com/zenn-user-upload/c397cd13c2e9-20221007.png)

変更が完了しましたので、画面右上の[適用して終了]をクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/4fbf4c2941b3-20221007.png)

[定義]のJSONやワークフロー図に[PutObject]が追加されました。

まだステートマシンの変更が保存されていませんので、画面右上の[保存]ボタンをクリックして保存しましょう。

![](https://storage.googleapis.com/zenn-user-upload/e23df4830fe0-20221007.png)

## Step3: Step Functionsを実際に動作させて、データを保存してみよう

実際にデータが保存されたかを試してみましょう。

Stripeで作成した支払いリンクから、サブスクリプションを申し込みします。

![](https://storage.googleapis.com/zenn-user-upload/2fac4f6334cf-20221005.png)

申し込み完了後、数秒〜1分程度でS3バケット内にJSONファイルが追加されます。

![](https://storage.googleapis.com/zenn-user-upload/afd9c416798d-20221007.png)

このように、Stripe Webhook / Amazon EventBridge / AWS StepFunctionsを組み合わせることで、コードを書かずにStripe内のイベントデータをAWS上で分析する下準備ができます。

## Step4: QuickSightで分析してみよう

※このステップは、当日AWS亀田さんによる解説を行います


**Memo: S3に保存したJSONデータを読むためのマニフェストJSONファイルサンプル**

```json
{
    "fileLocations": [
        {
            "URIPrefixes": [
                "https://s3.amazonaws.com/{BUCKET_NAME}/"
            ]
        }
    ],
    "globalUploadSettings": {
        "format": "JSON"
    }
}
```

## おさらい

- Step Functionsを利用して、StripeのイベントデータをS3へ保存できる
- AWSではS3にJSON, CSVデータを保存することでデータ分析ができる
- QuickSightで、データ分析・可視化に挑戦しよう

## [Advanced] 実運用を目指すための、チャレンジ項目

実際の分析シーンでは、さまざまなデータが必要となります。
Stripe WebhookとAmazon EventBridgeの設定をカスタマイズして、以下のデータに分析に挑戦してみましょう。

- 決済データ（payment_intent）
- 解約データ (customer.subscription.canceled)
