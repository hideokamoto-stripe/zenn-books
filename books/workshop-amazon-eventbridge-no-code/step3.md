---
title: "Step3: Stripe Customer Portalで、解約画面を用意しよう"
---

ホスティングサービスには、申し込みだけでなく解約時のサーバー削除フローも必要です。

StripeのサブスクリプションとCloudFormationを連携させた後、Step Functionsで解約フローを用意しましょう。

## サーバー情報とサブスクリプション情報の管理方法について

解約時の操作を自動化するには、CloudFormationで立ち上げたサーバーの情報と、Stripeが持つサブスクリプション情報を紐づける必要があります。

紐付け方にはさまざまな方法がありますが、今回はローコードで実現できるDynamoDBを利用した方法で進めます。

## Step1: DynamoDBとStep Functionsを利用して、サーバー情報とサブスクリプション情報を紐付けよう

DynamoDBへのアクセスは、引き続きStep Functionsを利用します。
### 1-1: DynamoDBでテーブルを作成しよう

マネージメントコンソール上部の検索フォームに[dynamodb]と入力しましょう。

検索結果、[DynamoDB]がでてきますので、クリックします。

![](https://storage.googleapis.com/zenn-user-upload/0da0600e3387-20221012.png)

DynamoDB管理画面に移動しました。左側メニューから[テーブル]を選択しましょう。

![](https://storage.googleapis.com/zenn-user-upload/ac0629ddd6d5-20221012.png)
[テーブル]右側にある[テーブルの作成]をクリックします。

![](https://storage.googleapis.com/zenn-user-upload/d6f48e7949c2-20221012.png)

テーブルを作成するには、[名前]と[パーテーションキー]の２つが必須です。

![](https://storage.googleapis.com/zenn-user-upload/16e52360dee8-20221012.png)

ここでは、以下のように入力しましょう。

- 名前: StripeWSContract
- パーテーションキー: subscription_id (文字列)

![](https://storage.googleapis.com/zenn-user-upload/43e3cb9d7de0-20221012.png)

テーブルの設定は、[デフォルト設定]で進めます。

![](https://storage.googleapis.com/zenn-user-upload/84e4b98326a0-20221012.png)

最後にページ下部の[テーブルの作成]をクリックしましょう。

作成が完了するまで、少し時間がかかることがあります。

![](https://storage.googleapis.com/zenn-user-upload/2a547a426202-20221012.png)

[正常に作成されました]のメッセージ表示が出るか、状態が[アクティブ]になっていれば、作成完了です。

![](https://storage.googleapis.com/zenn-user-upload/ac3e1c889519-20221012.png)
### 1-2: サーバー作成のStep Functionsステートマシンに、DynamoDBのPutItem操作を追加する

続いてステートマシンから、DynamoDBにデータを保存してみましょう。

Step1で作成した、Step Functionsのステートマシンを開きます。

![](https://storage.googleapis.com/zenn-user-upload/b81d681e7aa7-20221007.png)

右側にある[編集]ボタンをクリックし、編集画面を開きます。

[Workflow Studio]ボタンが、ワークフローを図示している部分にありますので、クリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/4eac31375004-20221007.png)

クリックすると、GUIでワークフローを編集できる画面が立ち上がります。

左側にある検索フォームに、[dynamodb putitem]と入力しましょう。

![](https://storage.googleapis.com/zenn-user-upload/7053c30f8026-20221012.png)

[PutItem]のカードを、ドラッグ＆ドロップで[DescribeStack]の前におきましょう。

![](https://storage.googleapis.com/zenn-user-upload/13528f249e43-20221012.png)

#### [設定]で入力値をを指定する

DBに保存するデータの内容は、右側の[設定]タブから指定できます。

![](https://storage.googleapis.com/zenn-user-upload/1a70e36c9138-20221012.png)

[統合タイプ]は[Optimized]のままにしましょう。

[APIパラメータ]には、以下のJSONを入力します。

```json
{
  "TableName": "StripeWSContract",
  "Item": {
    "subscription_id": {
      "S.$": "$.detail.data.object.id"
    },
    "stack_name": {
      "S.$": "$.stack.name"
    }
  }
}
```

#### [出力]で次のステップにわたす値を指定する

続いて[出力]タブを選びます。

今回は、「PutItemの結果」は次のステップで必要としません。

そのため、[ResultPathを使用して元の入力を出力に追加]をオンにし、[Discard result and keep original input]を選択しましょう。

![](https://storage.googleapis.com/zenn-user-upload/ccb245847f31-20221012.png)

変更が完了しましたので、画面右上の[適用して終了]をクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/4fbf4c2941b3-20221007.png)

[定義]のJSONやワークフロー図に[PutItem]が追加されました。

まだステートマシンの変更が保存されていませんので、画面右上の[保存]ボタンをクリックして保存しましょう。

![](https://storage.googleapis.com/zenn-user-upload/2a71e35182ec-20221012.png)

接続に成功していれば、Stripe Payment Links経由でのサブスクリプション申し込みで、DynamoDBにアイテムが保存されます。

![](https://storage.googleapis.com/zenn-user-upload/af7a3b9cea8a-20221012.png)

データは、テーブル[StripeWSContract]を開き、[テーブルアイテムの探索]をクリックすると確認できます。

![](https://storage.googleapis.com/zenn-user-upload/bd58fe720768-20221012.png)

## Step2: 解約ワークフローを実行する、Step Functionsステートマシンを作成しよう

解約時に参照するデータベースの用意ができました。

次は、このデータを利用してCloudFormationスタックを削除するステートマシンを追加しましょう。

Step1の手順と同様に、[ステートマシンの作成]ページを開きましょう。

![](https://storage.googleapis.com/zenn-user-upload/1e777ffb399f-20221012.png)

今回も、[コードでワークフローを記述]を選択します。

定義のJSONファイルは、以下をコピーアンドペーストしましょう。

```json
{
  "Comment": "A description of my state machine",
  "StartAt": "DynamoDB GetItem",
  "States": {
    "DynamoDB GetItem": {
      "Type": "Task",
      "Resource": "arn:aws:states:::dynamodb:getItem",
      "Parameters": {
        "TableName": "StripeWSContract",
        "Key": {
          "subscription_id": {
            "S.$": "$.detail.data.object.id"
          }
        }
      },
      "Next": "DeleteStack",
      "ResultSelector": {
        "subscription_id.$": "$.Item.subscription_id.S",
        "stack_name.$": "$.Item.stack_name.S"
      }
    },
    "DeleteStack": {
      "Type": "Task",
      "Next": "DynamoDB DeleteItem",
      "Parameters": {
        "StackName.$": "$.stack_name"
      },
      "Resource": "arn:aws:states:::aws-sdk:cloudformation:deleteStack",
      "ResultPath": null
    },
    "DynamoDB DeleteItem": {
      "Type": "Task",
      "Resource": "arn:aws:states:::dynamodb:deleteItem",
      "Parameters": {
        "TableName": "StripeWSContract",
        "Key": {
          "subscription_id": {
            "S.$": "$.subscription_id"
          }
        }
      },
      "Next": "Success"
    },
    "Success": {
      "Type": "Succeed"
    }
  }
}
```

これで、次の画像のようなフローのステートマシンが完成します。

![](https://storage.googleapis.com/zenn-user-upload/cbfe454dc44c-20221012.png)

[次へ]をクリックしましょう。

### 新しいステートマシンのロールは、step1のものを再利用

ステートマシンの名前と、ロールを設定しましょう。
![](https://storage.googleapis.com/zenn-user-upload/eb8f4fcc0a50-20221012.png)
名前は、`DeleteWPWorkflow`を入力します。

![](https://storage.googleapis.com/zenn-user-upload/b354c0e7f933-20221012.png)

ロールについては、step1で作成したロールを利用します。

![](https://storage.googleapis.com/zenn-user-upload/0d510727e9b1-20221012.png)

ワークフローが失敗した時の調査に備えて、ログは「ERROR」を指定します。
ロググループを新しく作成できます。ステートマシンの名前を利用できますので、デフォルトのまま使いましょう。

![](https://storage.googleapis.com/zenn-user-upload/ef4611b7bf95-20221012.png)

ページをスクロールして、[ステートマシンの作成]をクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/ad194ecb869b-20221004.png)

作成成功のメッセージが表示されれば、完了です。

![](https://storage.googleapis.com/zenn-user-upload/f41132855bd1-20221004.png)

## Step3: Amazon EventBridgeで、Stripe Webhookからの解約イベントを受け付けよう

続いて、作成したワークフローを実行するための、EventBridgeルールを追加します。

ページ上部の検索フォームに`eventbridge`を入力し、EventBridgeをクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/f5952e857871-20221005.png)

EventBridge管理画面では、左側メニューから[ルール]を選択します。

![](https://storage.googleapis.com/zenn-user-upload/13be672e6f8c-20221005.png)

### 3-1: ルールを作成しよう

イベントバスを`default`、ルール名を`stripe-delete-subscription`に設定して、ルールを作成しましょう。

今回は、イベントパターン設定でのサンプルイベント入力を省略します。

[イベントパターン]にて、フィルタリングルールをJSONで定義しましょう。
![](https://storage.googleapis.com/zenn-user-upload/b89f03e5af3c-20221005.png)

[イベントパターンのフォーム]から選択することも可能ですが、今回は[カスタムパターン（JSONエディタ）]タブを選びましょう。

![](https://storage.googleapis.com/zenn-user-upload/95067d4b45d4-20221005.png)

以下のJSONをエディタにコピー＆ペーストしましょう。

```json
{
  "source": [{
    "prefix": "stripe.com"
  }],
  "detail-type": ["customer.subscription.deleted"]
}
```

[次へ]をクリックして、ターゲットの設定に移ります。

### 3-2: ターゲットにStepFunctionsを指定しよう

続いて、イベントパターンと一致した場合に実行する処理を設定します。

[ターゲットタイプ]を[AWSのサービス]に設定しましょう。

![](https://storage.googleapis.com/zenn-user-upload/d9521ecb8937-20221005.png)

検索フォームに[step functions]を入力し、[Step functions ステートマシン]を選択します。

![](https://storage.googleapis.com/zenn-user-upload/d4e613654d44-20221005.png)

ステートマシンを選択する画面が表示されます、先ほど作成したステートマシン`DeleteWPWorkflow`を指定しましょう。

![](https://storage.googleapis.com/zenn-user-upload/bf033d882e59-20221005.png)

実行ロールで、[この特定のリソースについて新しいロールを作成]を選択し、[次へ]を選択します。

![](https://storage.googleapis.com/zenn-user-upload/16d6e369e0d4-20221005.png)

タグを設定する画面がありますが、今回は省略し、[次へ]を選びましょう。

![](https://storage.googleapis.com/zenn-user-upload/29705e718e54-20221005.png)

最後に設定の確認画面が表示されます。
設定内容を確認して、[ルールの作成]をクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/a90ce3524e1c-20221005.png)

ルールが作成されました。

![](https://storage.googleapis.com/zenn-user-upload/33d1bcfb2dac-20221005.png)

## Step4: Stripeダッシュボードで、解約ページとイベント送信準備を行う

最後に、Stripeダッシュボード上で解約ページの設定を行います。
### 4-1: サブスクリプション解約のWebhookを送信対象に追加する

Amazon EventBridgeでイベントを待ち受ける準備ができましたので、実際にイベントを送信しましょう。

Stripe Dashboardにて、[開発者]タブをクリックし、左側のメニューから[Webhook]を選択しましょう。

![](https://storage.googleapis.com/zenn-user-upload/f584383f5c0a-20221005.png)

Amazon EventBridgeのクイックスタートが完了していれば、**lambda-url**を含むURLのWebhookがすでに登録されています。

このWebhookをクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/4b9521a3d04e-20221005.png)

詳細ページにて、Stripeから送信するイベント内容を変更できます。

URL右側にある[...]をクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/864b6c12948d-20221005.png)

[詳細情報の更新]をクリックします。

![](https://storage.googleapis.com/zenn-user-upload/f6b912be4e77-20221005.png)

[送信イベント]にて、検索フォームに`customer.subscription.deleted`を入力し、一致するものをクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/5925cb724f21-20221012.png)

`customer.subscription.deleted`が追加された状態で、`エンドポイントを更新`を選択します。

![](https://storage.googleapis.com/zenn-user-upload/3e3ec557b7ba-20221012.png)

これでサブスクリプション解約時に、EventBridgeへイベントが送られるようになりました。
### 4-2: カスタマーポータルを有効化する

最後に、ユーザーがサブスクリプションを解約するためのページを用意します。

Stripe Dashboard右上の歯車アイコンをクリックし、[製品の設定]ページを開きましょう。
![](https://storage.googleapis.com/zenn-user-upload/b2723e609446-20221012.png)

[Billing]に[カスタマーポータル]がありますので、クリックします。
![](https://storage.googleapis.com/zenn-user-upload/a8cc277834b8-20221012.png)
カスタマーポータルの設定画面が開きました。
![](https://storage.googleapis.com/zenn-user-upload/5cf773b6db03-20221012.png)
今回はURLで直接アクセスできるようにしますので、[リンクからカスタマーポータルを起動する]にある、[テスト環境のリンクを有効化]をクリックしましょう。
![](https://storage.googleapis.com/zenn-user-upload/56084ffe6f66-20221012.png)
[カスタマーポータルリンク]が表示されればOKです。
![](https://storage.googleapis.com/zenn-user-upload/02cb8b2d1355-20221012.png)
### 4-3: カスタマーポータルで、プラン解約をサポートする

続いてカスタマーポータルで、ユーザーができる操作を変更します。

[サブスクリプション]をクリックし、設定項目を開きましょう。

[サブスクリプションをキャンセル]がオンになっていることを確認します。
![](https://storage.googleapis.com/zenn-user-upload/06db704c355b-20221012.png)
[設定をキャンセル]にて、[今すぐキャンセル]を選択しましょう。

![](https://storage.googleapis.com/zenn-user-upload/e0acfd35dccb-20221012.png)

[請求期間の終了時にキャンセル]を指定した場合、サブスクリプションの解約処理は現在のサイクルが終わってから実行されます。
実際のビジネスでは利用することもありますが、今回のワークショップでは、解約ワークフローが動作することをその場で確認するため、[今すぐキャンセル]の設定で進みます。

これで設定が完了です。
## 動作を確認する方法

解約ワークフローを試すには、カスタマーポータルにアクセスします。

![](https://storage.googleapis.com/zenn-user-upload/a0543a02582f-20221012.png)

以下の画像のメッセージが表示される場合があります。

![](https://storage.googleapis.com/zenn-user-upload/cee2038fbd55-20221012.png)

この場合は、Stripe内のCustomerデータを更新し、「ダッシュボードにログイン中のメールアドレス」に変更しましょう。
![](https://storage.googleapis.com/zenn-user-upload/f4f77bde8c44-20221012.png)

メールアドレスに対して、ログイン用のコードが送られてきます。
![](https://storage.googleapis.com/zenn-user-upload/2d95c3567468-20221012.png)

このコードを入力すると、マイページにアクセスできます。


![](https://storage.googleapis.com/zenn-user-upload/0fd244fd269e-20221012.png)

ここで、「プランをキャンセル」ボタンをクリックすると、サブスクリプションの解約ができます。

![](https://storage.googleapis.com/zenn-user-upload/6f3fe36189b6-20221012.png)

解約後は、StripeからAmazon EventBridgeにイベントが送られ、Step Functionsで解約ワークフローが起動します。

## おさらい

ここまでで、ユーザーがサブスクリプションを解約すると、自動的にサーバーが停止するワークフローが出来上がりました。

- DynamoDBなどのマネージド型DBを利用して、契約内容とシステムデータを連携できる
- StripeのCustomer Portalを利用して、サブスクリプションの管理や解約ができる
- イベントやタスクごとにStep Functionsのステートマシンを作成して、ローコードな開発とフローの可視化ができる

## [Advanced] 実運用を目指すための、チャレンジ項目

実運用では、サーバーを停止・削除する必要のあるイベントが少なくとももう１つ存在します。
それは、顧客が利用料金の支払いを行わなかったケースです。

Stripeでは、InvoiceまたはSubscriptionのイベントを利用して、未払いのサブスクリプションに対して処理を行えます。
今回のステートマシンを参考に、「未払いの顧客については、CloudFormationを自動削除する」ワークフローを構築してみましょう。
