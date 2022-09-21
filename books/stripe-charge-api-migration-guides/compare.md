---
title: "Charge/Token APIと、Payment Intentの違い"
---

作業前に、どのような点が変わっているかをおさらいします。

Payment Intent APIとToken / Charge APIでは、サーバー側でStripe APIを呼び出すタイミングが異なります。

これまでは、ユーザーが決済フォームの入力が完了してからStripe APIを呼び出していました。
しかし3Dセキュアやコンビニ決済・銀行振込など、リダイレクトまたは追加のユーザー操作が必要な決済方法を複数サポートするため、Payment Intentでは事前に呼び出す形を取ります。

事前にサーバー側でStripe APIを呼び出すため、フォームのSubmitイベント時にはStripe API呼び出す必要がなくなります。

|場面|Token / Charge API| Payment Intent API|
|:--|:--|:--|
|決済画面の表示前|なし|サーバー側で、Payment Intentを作成し、<br/>client_secretをフロントエンドに渡す|
|Stripe.jsでの決済画面表示|`<Element/>`の子孫要素に、<br/>`<CardElement/>`を含む`form`を作成する| 左と同じ|
|formをsubmitした時の操作|カード情報をToken化し、サーバー側で決済処理を完了する|サーバー側から取得した`client_secret`を使用し、フロントエンドで決済処理を完了する|
|在庫管理やCRM連携などの追加アクション|`charge.succeeded` Webhook| `payment_intent.succeeded` Webhook|

https://stripe.com/docs/payments/payment-intents/migration

Payment Intentの作成をフォームのsubmit時まで遅延させることも可能ですが、後半で紹介する`PaymentElement`への切り替えが難しくなるため、お勧めしません。