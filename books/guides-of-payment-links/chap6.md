---
title: "Payment Linksでできること・できないこと"
---

最後に、Payment Linksが向いているケースと、向いていないケースについて紹介します。

:::message
この情報は、**2023/05**時点でStripeが一般提供している機能をベースに紹介しています。
:::

## Stripe PaymentLinksが向いていないケース 

まずはあまり向いていないケースを3つ紹介します。


### 在庫を厳密に管理したいケース 
Stripeには在庫を管理する機能がデフォルトでは用意されていません。

在庫管理を支援するStripe Appsなどは公開されていますが、それでもRDBMSなどを利用した厳密なトランザクション管理が必要なケースには難しいと思われます。

https://marketplace.stripe.com/apps/stockify-manage-inventory

スクラッチでECサイトを開発されている場合は、Strip e Checkoutと在庫システムを連携させる方法をお勧めします。

### 商品数が多い、組み合わせ販売の種類が多いケース

Payment Linksでは、「事前に登録した商品」と「クロスセル登録されている商品」以外を注文できません。

そのため、商品の数や組み合わせパターンが多い場合には、Stripe Checkoutを利用して動的に決済フォームURLを生成しましょう。

### Customer情報を再利用したいケース

リピート注文が多いサイトなどでは、WordPressやfirebase / LINEのユーザー情報とStripeの注文履歴を連携する要件がでることがあります。

しかし2023/5時点のPayment Linksでは、「１度作成されたStripeのCustomerデータ」を使って決済フォームを用意することができません。

そのため、こちらもStripe Checkoutを利用する必要があります。

## Stripe PaymentLinksが向いているケース 
向いているケースとしては、次の例があります。

- 「立ち上げ初期またはプロトタイピングフェーズ」のプロダクトやサービス
- 個人でSNSなどを活用してビジネスを展開しているケース
- 開発工数をできるだけ少なくしたいプロジェクト

Stripeの特徴の１つは、「決済UIやフローを、ビジネスのフェーズに応じて段階的に変更できること」です。

Payment Linksで決済やサブスクリプションの受付を開始する形でローンチすることで、決済や請求管理周りの開発工数を大幅に削減できます。

そして上に紹介した「向いていない」要件が発生した場合は、Stripe Checkoutを利用することでローコードに切り替えや運用を継続できます。

WebhookやStripe Appsを活用する必要がある場合もありますが、顧客に提供するUIやシステムの開発・運用工数をセーブすることには大きく貢献できます。

https://stripe.com/docs/billing/subscriptions/webhooks

https://marketplace.stripe.com/

## Check point: 
- 「とりあえずやってみる」「素早い検証」などのプロトタイピング・MVP開発にPayment Linksは有効
- システム連携も、WebhookやStrpie Appsでカバーしやすい構成になっている
- ある程度売上や顧客ができたタイミングで、Stripe Checkoutすることでよりスケーラブルに