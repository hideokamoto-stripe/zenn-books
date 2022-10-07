---
title: "なぜCharge / Token / Source APIからPayment Intentsに切り替えるのか"
---

まず、Charge / Token / Source APIからPayment Intents / Setup Intentsに切り替える必要性について簡単に紹介します。

## 非推奨APIで、機能が限られている

最大の理由は、「すでに非推奨で、新しいAPI(Payment Intents / Setup Intents)の利用を推奨している」からです。

決済手段の多様化や不正利用対策の変化・各国の法律や規制の変化など、さまざまな要因で新しいAPIの提供を開始しました。

現在も非推奨APIを使い続けることは可能ですが、機能や決済手段の追加については、新しいAPIのみの提供となります。

https://stripe.com/docs/payments/older-apis

## 複数の決済方法や追加の認証をサポートできる

カード決済以外の支払い方法をサポートする場合、Charge / Token / Source APIがサポートしていないケースがあります。

日本では、コンビニ決済を利用する場合に、Payment Intents / Setup Intentsへの移行が必要です。

https://stripe.com/docs/payments/vouchers?locale=ja-JP

また、今後決済手段が追加された場合、同様に非推奨のAPIではサポートされない可能性があります。

## Strpe.jsの、Payment Elementが利用できる

Payment Elementは、1つの要素で複数の決済手段をサポートできる、埋め込み型の決済フォームです。

Payment IntentsまたはSetup Intentsのレスポンスを、Payment Elements作成時に渡すと、その顧客が利用可能な決済手段を自動で複数表示します。

![](https://storage.googleapis.com/zenn-user-upload/c067946efd67-20220916.png)

この機能を利用するには、Payment IntentsまたはSetup Intentsに切り替える必要があります。

## 3Dセキュア2に対応できる

Charge APIやToken API、Source APIでは、3Dセキュア2による認証が行えません。

これらのAPIを利用している場合、テスト環境にて、3Dセキュア2での認証テストに利用するカード番号`4000000000003220`を入力すると、エラーが発生します。

![](https://storage.googleapis.com/zenn-user-upload/061428c6d93f-20220916.png)

Payment IntentsやSetup Intentsを利用している場合、`4000000000003220`を入力すると、認証画面がページ内にポップアップ表示されます。

![](https://storage.googleapis.com/zenn-user-upload/47e110e9fd32-20220916.png)

Stripe.js側で画面表示の制御を行いますので、3Dセキュアに対応するための追加実装が必要なくなることもメリットです。