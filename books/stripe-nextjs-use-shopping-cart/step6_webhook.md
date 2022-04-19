---
title: "Webhookで注文内容を取得しよう"
---

ECサイトでは、注文内容を在庫システムや配送システムに共有する必要があります。

Stripeを利用する場合、注文データはWebhookで受け取ることができます。

最後のステップでは、Webhookの組み込み方法と、注文内容の取得方法を紹介します。


# おさらい

- Stripeの出来事はWebhookから受け取ることができる
- クロスセルや個数変更に備えて、Webhookで注文内容を取得しよう
- コンビニ決済など、一部の決済方法では注文と決済が別タイミングなことに注意

## 「早く終わった」という方のための、もう１ステップ

今回のワークショップでは、現在Stripeだけでは実装できない「在庫管理」についての説明を省略しています。

一方で、StripeのWebhookやuse-shopping-cartの`validateCartItems`などを利用することで、在庫システムとの連携に必要な情報やメソッドはある程度用意されています。

https://useshoppingcart.com/docs/usage/serverless/validate-cart-items

RDBMSやDynamoDB / Mongo DBなどのNoSQL DBなどを触ることができる方は、ぜひ在庫管理機能を追加することにチャレンジしてみてください。