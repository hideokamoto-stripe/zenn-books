---
title: "Stripeのデータを、分析用にS3へ保存しよう"
---

SaaSの運用や改善・アップデートには、サブスクリプションの契約情報などのデータ分析が必要になります。

ここでは、サブスクリプション申し込み時のデータをS3に保存し、QuickSightやAthenaで分析する準備を行います。

## Step1: 分析用データを保存するS3バケットを作成しよう

TBD
## Step2: Step　Functionsのステートマシンを編集し、S3にデータを保存しよう

TBD
## Step3: Step Functionsを実際に動作させて、データを保存してみよう

TBD
## Step4: QuickSightで分析してみよう

※このステップは、当日AWS亀田さんによる解説を行います


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
