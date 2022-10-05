---
title: "Step1: AWS Step Functionsで、WordPressサーバーを起動しよう"
---

まずはサービスのコア部分である、「サーバーのセットアップワークフロー」を構築しましょう。

## AWSマネージメントコンソールにログインしよう

まずはAWSマネージメントコンソールにログインしましょう。

[https://aws.amazon.com/jp/](https://aws.amazon.com/jp/?nc2=h_lg) から、[コンソールにサインイン]をクリックします。

![](https://storage.googleapis.com/zenn-user-upload/f29b1b0e8b2f-20221004.png)

その後、作成済みのIAMユーザーでログインしましょう。

![](https://storage.googleapis.com/zenn-user-upload/b52a40f52471-20221004.png)

ログインに成功すれば、マネージメントコンソールのTOPページが表示されます。

![](https://storage.googleapis.com/zenn-user-upload/c1546982db63-20221004.png)

### リージョンは、`us-east-1`

ワークショップを進める便宜上、リージョンは`米国東部（バージニア北部） us-east-1`を指定しましょう。

右上に「バージニア北部」と表示されていない場合、クリックして変更します。

![](https://storage.googleapis.com/zenn-user-upload/80ee21f8933a-20221004.png)

## Step1: EC2インスタンスのキーペアを作成しよう

今回作成するWordPressサーバーには、SSHやSFTPでアクセスするためのキーペアが必要です。

ワークショップ用にダミーのキーペアを作成しておきましょう。

ページ上部の検索フォームに、[ec2]と入力しましょう。

![](https://storage.googleapis.com/zenn-user-upload/7abb8659aea4-20221004.png)

[EC2]を選択して、EC2管理画面へ移動します。

![](https://storage.googleapis.com/zenn-user-upload/2656a58557fa-20221004.png)

左側のメニューから、[キーペア]を選択しましょう。

その後、画面右上にある[キーペアを作成]ボタンをクリックします。

キーペア作成画面では、名前の設定以外はデフォルトのままにしましょう。

![](https://storage.googleapis.com/zenn-user-upload/d0327487c70f-20221004.png)

名前も、[for-stripe-workshop]など、後から用途を思い出せるものにします。

名前を入力後、[キーペアを作成]ボタンをクリックして、作成を完了させましょう。

![](https://storage.googleapis.com/zenn-user-upload/74664209010d-20221004.png)

[正常に作成されました]メッセージが表示されれば、準備完了です。

## Step1: AWS Step Functionsでステートマシンを作成しよう

ここからは、ノーコード・ローコードにワークフローを構築できる製品「AWS StepFunctions」を利用します。

ページ上部の検索フォームに、[step functions]と入力しましょう。

![](https://storage.googleapis.com/zenn-user-upload/2be3ac826888-20221004.png)

[Step Functions]が検索結果に表示されますので、クリックします。

![](https://storage.googleapis.com/zenn-user-upload/b071a0903003-20221004.png)

ページ左側にある[ハンバーガーボタン]（横向きの線が３本あるボタン）をクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/d636eb6775b0-20221004.png)

[ステートマシン]を選びます。

![](https://storage.googleapis.com/zenn-user-upload/75fd317c8b1a-20221004.png)

[ステートマシンを作成]ボタンをクリックすると、ステートマシンの作成を開始できます。

### 1-1: ステートマシンの作成方法・タイプを選ぶ

まずはステートマシンの作成方法とタイプを選びましょう。

![](https://storage.googleapis.com/zenn-user-upload/95ff1033630e-20221004.png)

今回のサンプルでは、[コードでワークフローを記述]と[標準]を選択しましょう。

![](https://storage.googleapis.com/zenn-user-upload/038482ccc0bf-20221004.png)

画面下部に、コードを入力する画面が表示されました。

![](https://storage.googleapis.com/zenn-user-upload/046e1440592a-20221004.png)

左側のエディタ画面に、以下のJSONをコピーアンドペーストしましょう。

```json
{
  "Comment": "A description of my state machine",
  "StartAt": "CreateStack",
  "States": {
    "CreateStack": {
      "Type": "Task",
      "Parameters": {
        "StackName.$": "States.Format('LAMP-{}', $.name)",
        "TemplateURL": "https://s3-external-1.amazonaws.com/cloudformation-templates-us-east-1/WordPress_Single_Instance.template",
        "Parameters": [
          {
            "ParameterKey": "KeyName",
            "ParameterValue": "for-stripe-workshop"
          },
          {
            "ParameterKey": "DBName",
            "ParameterValue": "MyDatabase"
          },
          {
            "ParameterKey": "DBPassword",
            "ParameterValue": "userDBPassword123"
          },
          {
            "ParameterKey": "DBRootPassword",
            "ParameterValue": "rootDBPassword123"
          },
          {
            "ParameterKey": "DBUser",
            "ParameterValue": "user"
          },
          {
            "ParameterKey": "InstanceType",
            "ParameterValue": "t2.micro"
          },
          {
            "ParameterKey": "SSHLocation",
            "ParameterValue": "0.0.0.0/0"
          }
        ]
      },
      "Resource": "arn:aws:states:::aws-sdk:cloudformation:createStack",
      "Next": "DescribeStacks",
      "ResultPath": "$.result"
    },
    "DescribeStacks": {
      "Type": "Task",
      "Parameters": {
        "StackName.$": "$.result.StackId"
      },
      "Resource": "arn:aws:states:::aws-sdk:cloudformation:describeStacks",
      "Next": "Choice",
      "ResultSelector": {
        "name.$": "$.Stacks[0].StackName",
        "status.$": "$.Stacks[0].StackStatus"
      },
      "ResultPath": "$.stack"
    },
    "Choice": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.stack.status",
          "StringEquals": "CREATE_IN_PROGRESS",
          "Next": "Wait"
        },
        {
          "Variable": "$.stack.status",
          "StringEquals": "CREATE_COMPLETE",
          "Next": "Success"
        }
      ],
      "Default": "Fail"
    },
    "Fail": {
      "Type": "Fail"
    },
    "Wait": {
      "Type": "Wait",
      "Seconds": 30,
      "Next": "DescribeStacks (1)"
    },
    "DescribeStacks (1)": {
      "Type": "Task",
      "Parameters": {
        "StackName.$": "$.stack.name"
      },
      "Resource": "arn:aws:states:::aws-sdk:cloudformation:describeStacks",
      "Next": "Choice",
      "ResultSelector": {
        "name.$": "$.Stacks[0].StackName",
        "status.$": "$.Stacks[0].StackStatus"
      },
      "ResultPath": "$.stack"
    },
    "Success": {
      "Type": "Succeed"
    }
  }
}
```

Step Functionsでは、「ビジュアルエディタを利用したワークフローの構築」だけでなく、このようにJSONを利用してワークフローを複製・共有できます。

右側のワークフローが、以下の画像のように変わればOKです。

![](https://storage.googleapis.com/zenn-user-upload/039ddd69ebbc-20221004.png)

[次へ]をクリックしましょう。

### 1-2: ステートマシンの詳細設定を行おう

続いて、ステートマシンの名前やログの設定を行います。

名前は、`CreateWPWorkflow`を入力してください。

また、[アクセス許可]は[新子ロールの作成]を選択しましょう。

![](https://storage.googleapis.com/zenn-user-upload/a50ea13a54b6-20221004.png)

ワークフローが失敗した時の調査に備えて、ログは「ERROR」を指定します。

![](https://storage.googleapis.com/zenn-user-upload/3e53fef48c69-20221004.png)

ロググループを新しく作成できます。ステートマシンの名前を利用できますので、デフォルトのまま使いましょう。

![](https://storage.googleapis.com/zenn-user-upload/627c464d0b91-20221004.png)

ページをスクロールして、[ステートマシンの作成]をクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/ad194ecb869b-20221004.png)

作成成功のメッセージが表示されれば、完了です。

![](https://storage.googleapis.com/zenn-user-upload/f41132855bd1-20221004.png)

## Step2: IAMロールを更新して、CloudFormation経由でのEC2インスタンス作成準備をしよう

ステートマシンの作成には成功しました。

ですが作成したステートマシンの画面には警告メッセージが表示されています。

![](https://storage.googleapis.com/zenn-user-upload/e99d16564dee-20221004.png)

これは、ステートマシンがAWSのリソースを操作するために、必要な権限を持っていないことを示しています。

[IAMでロールを編集]ボタンをクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/d22153f92f3c-20221004.png)

IAMロールの編集画面に移動しました。

[許可ポリシー]エリアの右側にある[許可を追加]をクリックし、[インラインポリシーを作成]を選択しましょう。

![](https://storage.googleapis.com/zenn-user-upload/179e835335ed-20221004.png)

ポリシー作成画面が表示されます。

![](https://storage.googleapis.com/zenn-user-upload/ea051d9737ac-20221004.png)

今回はここでまとめて権限設定を行うため、[JSON]タブを選択しましょう。

![](https://storage.googleapis.com/zenn-user-upload/dd81f5e8ac95-20221004.png)

JSON入力欄に、以下のコードを上書き入力します。

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "lambda:InvokeFunction",
                "ec2:AuthorizeSecurityGroupIngress",
                "ec2:DescribeInstances",
                "ec2:StartInstances",
                "ec2:CreateSecurityGroup",
                "ec2:DescribeKeyPairs",
                "ec2:TerminateInstances",
                "ec2:RunInstances",
                "ec2:StopInstances",
                "ec2:DescribeSecurityGroups",
                "cloudformation:DescribeStackResource",
                "cloudformation:DescribeStackEvents",
                "cloudformation:UpdateStack",
                "cloudformation:ListStackResources",
                "cloudformation:DescribeStacks",
                "cloudformation:CreateStack",
                "cloudformation:DeleteStack",
                "ec2:DeleteSecurityGroup"
            ],
            "Resource": "*"
        }
    ]
}
```
![](https://storage.googleapis.com/zenn-user-upload/a83662dba301-20221004.png)

[ポリシーの確認]をクリックすると、権限の確認画面が表示されます。

![](https://storage.googleapis.com/zenn-user-upload/c3a71eaafd57-20221004.png)

名前を指定できますので、[stripeWorkshopPolicy]を入力しましょう。

[ポリシーの作成]をクリックして、作業完了です。

## Step3: 作成したワークフローを動かしてみよう

権限設定が完了しましたので、一度実際にワークフローを動かしてみましょう。

Step Functionsのステートマシン画面に戻ります。

![](https://storage.googleapis.com/zenn-user-upload/e7d8ca06906e-20221004.png)

[実行]セクション右側にある、[実行の開始]ボタンをクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/bd9fd0cf3f41-20221004.png)

入力データを指定して、ワークフローを手動実行する画面が開きます。

以下のJSONを[入力]部分に上書き入力しましょう。

```json
{
    "name": "helloWP"
}
```

入力できれば、[実行の開始]ボタンをクリックします。

![](https://storage.googleapis.com/zenn-user-upload/e4d7aa038252-20221004.png)

[グラフインスペクター]にワークフローが色付きで表示されれば、実行成功です。

![](https://storage.googleapis.com/zenn-user-upload/f4ebebb67921-20221004.png)

WordPressが立ち上がるまで、数分かかります。

![](https://storage.googleapis.com/zenn-user-upload/7dd48c9787fa-20221004.png)

[SUCCESS]が緑色になれば、ワークフロー完了です。

### CloudFormation管理画面から、WordPressにアクセスしよう

作成したサーバーへ実際にアクセスしてみましょう。

まずはCloudFormation管理画面へ移動します。

上部の検索フォームで[cloudformation]と入力し、[CloudFormation]をクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/9a1bb0ff7756-20221004.png)

一覧画面に`LAMP-helloWP`が表示されていますので、クリックしましょう。

今回のサンプルでは、`LAMP-{実行時に設定した名前}`でリソースが作成されます。

![](https://storage.googleapis.com/zenn-user-upload/32cec4ba87e9-20221004.png)

詳細画面で、[出力]タブを選びましょう。`WebsiteURL`にURLが表示されています。

![](https://storage.googleapis.com/zenn-user-upload/9b396f92b880-20221004.png)

クリックすると、WordPressのインストール画面に移動します。

![](https://storage.googleapis.com/zenn-user-upload/9ae7c4613e58-20221004.png)

## おさらい

- AWS StepFunctionsを利用して、サーバーのセットアップなどのワークフローが実現できる
- ChoiceやWaitを利用して、時間のかかるタスクを待機することができる
- JSONを利用して、Step Functionsのステートマシン定義をシェアしたり、コード管理できる


## [Advanced] 実運用を目指すための、チャレンジ項目

AWSが用意しているサンプルのCloudFormationテンプレートを利用して、簡単にWordPressサーバーを立ち上げるワークフローを構築しました。

実際にシステムとして利用するには、このステートマシンにさまざまなタスクを追加する必要があります。

以下にアイディアやヒントを用意しましたので、時間に余裕がある方はぜひ挑戦して、ご自身のブログやコメントなどでお知らせください。

- Step FunctionsとS3を利用して、EC2のキーペア作成と共有を自動化しよう
- Amazon SESやSNSで、顧客または社内にサーバー立ち上げの成功・失敗を通知しよう
- CloudFormationのテンプレートを別のものや自作品に差し替えてみよう