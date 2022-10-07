---
title: "Stripe Customer Portalで、解約画面を用意しよう"
---

ホスティングサービスには、申し込みだけでなく解約時のサーバー削除フローも必要です。

StripeのサブスクリプションとCloudFormationを連携させた後、Step Functionsで解約フローを用意しましょう。

## Step1: AWS LambdaでStripeのサブスクリプション情報を更新する関数を用意しよう

## Step2: サーバー作成のStep Functionsステートマシンに、サブスクリプション更新Lambdaを追加しよう

## Step3: サーバー作成のステートマシンをもとに、解約フローのステートマシンを作成しよう

## Step4: Stripe Customer Portalで、サブスクリプション解約を有効化しよう

## Step5: Stripe Webhookで、サブスクリプション解約イベントもAmazon EventBridgeに送信しよう

## Step6: Amazon EventBridgeで、サブスクリプション解約のワークフローを設定しよう


