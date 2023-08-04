---
title: "Step 2: Next.js REST API の開発と注文処理の統合"
---

Step1で、商品登録と表示、そして注文ページへの動線まで用意できました。

e-commerceサイトでは、顧客が商品を注文した後に、商品の準備や発送を行う必要があります。

社内の発送システムに注文情報を送信するため、Stripeで受け付けた注文を外部システムに連携する「Webhook API」を開発しましょう。

:::message
**「社内システムとの連携」についての補足**
ワークショップでは、「Stripeから送信された注文情報を受け取るAPI」までを開発します。
「注文情報を受け取る仕組みまでを用意し、実際の連携作業は別チームが担当する」ような開発フロー・・・ということで進めましょう。
:::

## Next.js(v13 App Router)で、REST APIを追加する

## Stripe CLIで、StripeのWebhookイベントをNext.jsに中継する


## 注文内容を表示する処理を実装しよう

## おさらい

- Next.js(v13)のApp Routerでは、`app/[パス名]/route.ts`でAPIを追加できる
- どのメソッドをサポートするかは、`export function [メソッド名]`で定義できる
- Stripe CLIを使えば、StripeのWebhookイベントをローカル環境に転送できる
