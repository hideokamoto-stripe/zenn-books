---
title: "Chapter4: Stripeのデータを、分析用にS3へ保存しよう"
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

データをS3に投入しましたので、QuickSight可視化しましょう。
### 4-1: QuickSightをセットアップする

QuickSightのセットアップを行いましょう。

上部の検索フォームで、[Quicksight]を入力し、[QuickSight]をクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/67da89d6576b-20221013.png)

アカウントを作成したことがない場合、作成画面が表示されます。
すでにある方は、4-2へスキップしましょう。

![](https://storage.googleapis.com/zenn-user-upload/646cf22ee9ad-20221013.png)

アカウントの種類を選びます。
ここでは、[standard]を選びましょう。

![](https://storage.googleapis.com/zenn-user-upload/09638a143c2d-20221013.png)

デフォルト設定のまま、進めます。

![](https://storage.googleapis.com/zenn-user-upload/b0786187ef8f-20221013.png)

アカウント名と、メールアドレスを入力しましょう。

![](https://storage.googleapis.com/zenn-user-upload/25377010c7d9-20221013.png)

データソースにするAWSリソースを選択します。

![](https://storage.googleapis.com/zenn-user-upload/ddf91479354b-20221013.png)

デフォルトではS3が選ばれていませんので、追加しましょう。

![](https://storage.googleapis.com/zenn-user-upload/fd4c29579994-20221013.png)

バケット名を指定する必要がありますので、先ほど作成したバケットを指定しましょう。

![](https://storage.googleapis.com/zenn-user-upload/7abe9de4ca5d-20221013.png)

[Finish]を押すと、S3が選択状態に変わります。

![](https://storage.googleapis.com/zenn-user-upload/0dba6394d056-20221013.png)

[Finish]を押して、作成を始めましょう。

![](https://storage.googleapis.com/zenn-user-upload/26e97deff410-20221013.png)

作成に成功したら、[Go to Amazon QuickSight]をクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/ae559632cd5c-20221013.png)

### 4-2: S3からデータセットを読み込みしよう

ダッシュボードにアクセスできましたので、S3のデータを「データセット」として読み込みましょう。

画面左側の[データセット]をクリックします。

![](https://storage.googleapis.com/zenn-user-upload/8fcbf2c5d381-20221013.png)

画面右上にある、[新しいデータセット]をクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/d4de6d303a19-20221013.png)

データソースを指定します。ここでは[S3]を選びましょう。

![](https://storage.googleapis.com/zenn-user-upload/cf22da480e44-20221013.png)

データソース名と、マニフェストファイルを指定します。

データソース名は、`stripesubscription`など後でわかりやすい名前をつけましょう。

![](https://storage.googleapis.com/zenn-user-upload/f9c5362b7800-20221013.png)

マニフェストファイルは、手元で作成する必要があります。

`manifest.json`ファイルを作成し、以下のJSONを入力しましょう。
　
**manifest.json**
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

`{BUCKET_NAME}`を、先ほど作成したS3バケット名に設定しましょう。

[アップロード]を選んで、作成したJSONファイルをアップロードすればOKです。

![](https://storage.googleapis.com/zenn-user-upload/a09eb8826f5e-20221013.png)

[接続]をクリックすると、データセット作成完了メッセージが表示されます。

![](https://storage.googleapis.com/zenn-user-upload/62c6328704e3-20221013.png)
### 4-3: 取り込みしたデータを可視化しよう

※このステップは、当日AWS亀田さんによる解説を行います
## おさらい

- Step Functionsを利用して、StripeのイベントデータをS3へ保存できる
- AWSではS3にJSON, CSVデータを保存することでデータ分析ができる
- QuickSightで、データ分析・可視化に挑戦しよう

## [Advanced] 実運用を目指すための、チャレンジ項目

実際の分析シーンでは、さまざまなデータが必要となります。
Stripe WebhookとAmazon EventBridgeの設定をカスタマイズして、以下のデータに分析に挑戦してみましょう。

- 決済データ（payment_intent）
- 解約データ (customer.subscription.canceled)
