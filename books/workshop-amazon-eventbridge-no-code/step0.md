---
title: "はじめに"
---

このワークショップ資料は、JP_Stripesで開催された「AWSでサーバーレスなSaaSバックエンド開発ワークショップ」の後編資料です。
Amazon EventBridgeを利用したStripeのWebhookイベント受信方法については、前編資料をご覧ください。
https://前編資料のURL

## 体験できること
- AWSを利用して、最小限のコードで、SaaSのバックエンドシステムを構築できること
- StepFunctionsでさまざまなワークフローが構築できること
- StripeとEventBridgeを連携させると、より簡単にサブスクリプションのバックエンドが構築できること

## ワークショップに必要なもの

- 以下の条件を満たすAWSアカウント
  - マネージメントコンソールにログインできること
  - Administratorロールまたはそれに準ずる権限を持つアカウントでログインできること
- Stripeアカウント
  - 本番利用申請は不要です
  - コンタミ・誤デプロイなどを防ぐため、ワークショップ専用のアカウントの用意を推奨します


## ワークショップの注意点

- CloudFormationを利用して、EC2インスタンス(`t3.micro`)を起動します。  
無料枠をすでに消化している場合、EC2インスタンスの利用料金が発生しますのでご注意ください。
- StepFunctionsやLambdaなどについても、そのほかのサービスで無料利用枠をすでに消化している場合、所定の利用料金が発生します。


## ワークショップの流れ

- Lambda関数を用意する（Stripe APIをSDKなしで呼び出す方法）
- StepFunctionsのステートマシンを作成する
  - 作成したLambdaを実行する
  - CloudFormationを起動する
  - CloudFormationのステータス監視システムを作成する
  - [optional] 作成失敗時に解約・返金する
- EventBridgeと連携する
  - フィルター
  - トランスフォーマー
- Payment Linksでサブスクリプションを申し込みする
- 解約時のステートマシンを作成する
  - CloudFormationを削除する
  - CloudFormationのステータス監視システムを作成する
  - EventBridgeと連携する
- Customer Portalで解約操作を試す
- SaaSの分析
  - S3にイベントデータを保存する
  - Stripe Shell（ブラウザ）でデモデータを生成する
  - QuickSightでデータを可視化する