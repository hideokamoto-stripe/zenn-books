---
title: "Chapter3: Stripe Payment Linksで、サーバー申し込みフォームを用意しよう"
---

CloudFormation経由でサーバーを起動するワークフローが用意できました。

続いて、サーバー起動のワークフローを開始するための「申し込みフォーム」をStripeで用意しましょう。

## Stripeダッシュボードにログインしよう

まずはStripeダッシュボードにログインしましょう。

![](https://storage.googleapis.com/zenn-user-upload/66f726dd3f1d-20221005.png)
## Step1: Payment Linksでサーバー申し込みフォームを作成しよう

早速申し込みフォームを作成しましょう。

ダッシュボード右上の[作成]をクリックし、[支払いリンク]を選択します。
もしくは、キーボードで[c]->[l]の順にキーを押しましょう。

![](https://storage.googleapis.com/zenn-user-upload/0c2890c4931c-20221005.png)

支払いリンク（Payment Links）作成画面が開きます。

![](https://storage.googleapis.com/zenn-user-upload/1ccee5132957-20221005.png)

### 1-1: 支払いタイプを[商品またはサブスクリプション]に設定する

Payment Linksでは、NPO団体への寄付のような「買い手が価格を決める」方法を選ぶこともできます。

[タイプ]で[顧客が支払い金額を選択する]を選択することでモードを変更できますが、今回はホスティングサービスですので、[商品またはサブスクリプション]のまま進めます。

![](https://storage.googleapis.com/zenn-user-upload/179a13eeaf0d-20221005.png)

### 1-2: 申し込みするプラン情報を登録する

続いて申し込みするプランを登録しましょう。

[商品]の入力欄を選択すると、[新しい商品を追加]が表示されますので、クリックします。

![](https://storage.googleapis.com/zenn-user-upload/d7bbbfb2d053-20221005.png)

Stripeに商品や料金プランを登録する画面が立ち上がります。

![](https://storage.googleapis.com/zenn-user-upload/8729b43a05d3-20221005.png)

ここでは、以下のように情報を入力しましょう。

|項目名|値|
|:--|:--|
|名前|WordPressホスティング|
|説明|（空欄）|
|画像|（空欄）|
|価格|4000|
|継続・一括|継続|
|請求期間|月次|

[商品を追加]ボタンをクリックして、登録を完了しましょう。

登録に成功していれば、[商品]欄やプレビュー画面に[WordPressホスティング]が表示されています。

![](https://storage.googleapis.com/zenn-user-upload/5dec2769a7ef-20221005.png)

### 1-3: 支払リンクの作成を完了させよう

ページ上部の[次へ]をクリックすると、確認画面またはConnectを利用する場合の設定画面が表示されます。

![](https://storage.googleapis.com/zenn-user-upload/0cf5347a61b7-20221005.png)

[連結アカウントに支払いを分割]が表示され、チェックボックスがオンになっている場合は、解除しましょう。

![](https://storage.googleapis.com/zenn-user-upload/152422c2f7f8-20221005.png)

[リンクを作成]をクリックすると、作成が完了します。

![](https://storage.googleapis.com/zenn-user-upload/24c683987f68-20221005.png)

発行されたURLやQRコードを使って、サービスの申し込みページを簡単に顧客へ共有することができます。

## Step2: Amazon EventBridgeで、サブスクリプション申し込み時のサーバー起動ワークフローを設定しよう

支払リンクの作成が完了しましたので、StripeのイベントとStepFunctionsを紐づける作業をEventBridgeで行いましょう。

### 2-1: Amazon EventBridgeコンソールに移動する

紐付け作業は、AWSのマネージメントコンソール上で行います。

ページ上部の検索フォームに`eventbridge`を入力し、EventBridgeをクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/f5952e857871-20221005.png)

EventBridge管理画面では、左側メニューから[ルール]を選択します。

![](https://storage.googleapis.com/zenn-user-upload/13be672e6f8c-20221005.png)


### 2-2: ルールを作成する

[イベントバス]が[default]になっていることを確認しましょう。

![](https://storage.googleapis.com/zenn-user-upload/74ed33db07c1-20221005.png)

[ルール]の右側にある[ルールを作成]ボタンをクリックします。

![](https://storage.googleapis.com/zenn-user-upload/8d19c473e972-20221005.png)

ルール作成のウィザードが立ち上がりました。

![](https://storage.googleapis.com/zenn-user-upload/000647e2726b-20221005.png)

名前には、`stripe-craete-subscription`を入力しましょう。

![](https://storage.googleapis.com/zenn-user-upload/91f926218547-20221005.png)

イベントバスが`default`になっていることを確認します。

![](https://storage.googleapis.com/zenn-user-upload/7c111984cdb9-20221005.png)

`ルールタイプ`は、`イベントパターンを持つルール`を指定しましょう。

![](https://storage.googleapis.com/zenn-user-upload/33a37e90e58d-20221005.png)

[次へ]をクリックして、イベントパターンの設定に移ります。

### 2-3: イベントパターン設定で、サブスクリプション申し込みのみ受け付けるルールを設定しよう

イベントパターンの設定では、「どのようなイベントを処理するか」を設定します。

今回は、「Stripeから送信された、サブスクリプションの申し込みイベント」のみを処理するルールを作りましょう。

イベントソースでは、`AWSイベントまたはEventBridgeパートナーイベント`を選択します。

![](https://storage.googleapis.com/zenn-user-upload/5a8843164ea7-20221005.png)

続いてサンプルイベントを設定しましょう。
`ご自身名前を入力`を選択し、以下のJSONをコピー&ペーストします。

```json
{
  "version": "0",
  "id": "fbf2573e-ed83-d8c4-9d68-8cf7eee6aec9",
  "detail-type": "customer.subscription.created",
  "source": "stripe.com",
  "account": "12345678921",
  "time": "2022-08-08T19:58:17Z",
  "region": "us-east-1",
  "resources": [],
  "detail": {
    "id": "xxx_3LUc5bBvzWSP8ADK1CqxxxFe",
    "object": "event",
    "api_version": "2020-08-27; orders_beta=v4",
    "created": 1659988697,
    "data": {
      "object": {
        "id": "sub_1LpVBNL6R1kGwUF4LQjOebhW",
        "object": "subscription",
        "application": null,
        "application_fee_percent": null,
        "automatic_tax": {
          "enabled": false
        },
        "billing_cycle_anchor": 1664967033,
        "billing_thresholds": null,
        "cancel_at": null,
        "cancel_at_period_end": false,
        "canceled_at": null,
        "collection_method": "charge_automatically",
        "created": 1664967033,
        "currency": "usd",
        "current_period_end": 1667645433,
        "current_period_start": 1664967033,
        "customer": "cus_MYcQXlENmEltqQ",
        "days_until_due": null,
        "default_payment_method": null,
        "default_source": null,
        "default_tax_rates": [],
        "description": null,
        "discount": null,
        "ended_at": null,
        "items": {
          "object": "list",
          "data": [{
            "id": "si_MYcQhfIgD0z1kb",
            "object": "subscription_item",
            "billing_thresholds": null,
            "created": 1664967034,
            "metadata": {},
            "plan": {
              "id": "price_1LpVBML6R1kGwUF4BUea6LSO",
              "object": "plan",
              "active": true,
              "aggregate_usage": null,
              "amount": 1500,
              "amount_decimal": "1500",
              "billing_scheme": "per_unit",
              "created": 1664967032,
              "currency": "usd",
              "interval": "month",
              "interval_count": 1,
              "livemode": false,
              "metadata": {},
              "nickname": null,
              "product": "prod_MYcQVAEEZzHfsq",
              "tiers_mode": null,
              "transform_usage": null,
              "trial_period_days": null,
              "usage_type": "licensed"
            },
            "price": {
              "id": "price_1LpVBML6R1kGwUF4BUea6LSO",
              "object": "price",
              "active": true,
              "billing_scheme": "per_unit",
              "created": 1664967032,
              "currency": "usd",
              "custom_unit_amount": null,
              "livemode": false,
              "lookup_key": null,
              "metadata": {},
              "nickname": null,
              "product": "prod_MYcQVAEEZzHfsq",
              "recurring": {
                "aggregate_usage": null,
                "interval": "month",
                "interval_count": 1,
                "trial_period_days": null,
                "usage_type": "licensed"
              },
              "tax_behavior": "unspecified",
              "tiers_mode": null,
              "transform_quantity": null,
              "type": "recurring",
              "unit_amount": 1500,
              "unit_amount_decimal": "1500"
            },
            "quantity": 1,
            "subscription": "sub_1LpVBNL6R1kGwUF4LQjOebhW",
            "tax_rates": []
          }],
          "has_more": false,
          "total_count": 1,
          "url": "/v1/subscription_items?subscription=sub_1LpVBNL6R1kGwUF4LQjOebhW"
        },
        "latest_invoice": "in_1LpVBNL6R1kGwUF43rS4NwaO",
        "livemode": false,
        "metadata": {},
        "next_pending_invoice_item_invoice": null,
        "pause_collection": null,
        "payment_settings": {
          "payment_method_options": null,
          "payment_method_types": null,
          "save_default_payment_method": "off"
        },
        "pending_invoice_item_interval": null,
        "pending_setup_intent": null,
        "pending_update": null,
        "plan": {
          "id": "price_1LpVBML6R1kGwUF4BUea6LSO",
          "object": "plan",
          "active": true,
          "aggregate_usage": null,
          "amount": 1500,
          "amount_decimal": "1500",
          "billing_scheme": "per_unit",
          "created": 1664967032,
          "currency": "usd",
          "interval": "month",
          "interval_count": 1,
          "livemode": false,
          "metadata": {},
          "nickname": null,
          "product": "prod_MYcQVAEEZzHfsq",
          "tiers_mode": null,
          "transform_usage": null,
          "trial_period_days": null,
          "usage_type": "licensed"
        },
        "quantity": 1,
        "schedule": null,
        "start_date": 1664967033,
        "status": "active",
        "test_clock": null,
        "transfer_data": null,
        "trial_end": null,
        "trial_start": null
      }

    },
    "livemode": false,
    "pending_webhooks": 3,
    "request": {
      "id": "req_xxxxxxxxx",
      "idempotency_key": "xxxxxx-e095-4b13-a5bc-xxxxxxxxx"
    },
    "type": "customer.subscription.created"
  }
}
```

![](https://storage.googleapis.com/zenn-user-upload/b8528dbe2866-20221005.png)

[イベントパターン]にて、フィルタリングルールをJSONで定義します。
![](https://storage.googleapis.com/zenn-user-upload/b89f03e5af3c-20221005.png)

[イベントパターンのフォーム]から選択することも可能ですが、今回は[カスタムパターン（JSONエディタ）]タブを選びましょう。

![](https://storage.googleapis.com/zenn-user-upload/95067d4b45d4-20221005.png)

以下のJSONをエディタにコピー＆ペーストしましょう。

```json
{
  "source": [{
    "prefix": "stripe.com"
  }],
  "detail-type": ["customer.subscription.created"]
}
```
![](https://storage.googleapis.com/zenn-user-upload/15eca6c2851e-20221005.png)

[テストパターン]ボタンをクリックすることで、サンプルイベントで入力したJSONがフィルターに一致するか否かをテストできます。

![](https://storage.googleapis.com/zenn-user-upload/6a72427d7897-20221005.png)

上の画像のように、緑色のメッセージが表示されていればOKです。

赤色が表示された場合は、「サンプルイベントのJSONデータが、フィルターに一致しない」ことを示しています。
サンプルイベントのデータまたはフィルターのJSONを見直して再度試しましょう。


![](https://storage.googleapis.com/zenn-user-upload/513401e06461-20221005.png)

フィルターの設定に成功すれば、[次へ]をクリックしましょう。
### 2-4: ターゲットにStepFunctionsを指定しよう

続いて、イベントパターンと一致した場合に実行する処理を設定します。

[ターゲットタイプ]を[AWSのサービス]に設定しましょう。

![](https://storage.googleapis.com/zenn-user-upload/d9521ecb8937-20221005.png)

検索フォームに[step functions]を入力し、[Step functions ステートマシン]を選択します。

![](https://storage.googleapis.com/zenn-user-upload/d4e613654d44-20221005.png)

ステートマシンを選択する画面が表示されます、前のステップで作成したステートマシンを指定しましょう。

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

これでStripeとStepFunctionsのワークフローがつながりました。
### 3-5: 実際に動かしてみよう

最後に、実際にサブスクリプションを申し込みしてみましょう。

Stripeダッシュボードにて、支払いリンクをコピーし、ブラウザに貼り付けて移動しましょう。

![](https://storage.googleapis.com/zenn-user-upload/24c683987f68-20221005.png)

決済フォームが表示されます。

![](https://storage.googleapis.com/zenn-user-upload/2fac4f6334cf-20221005.png)

テスト環境では、以下のカード情報がテスト用途で利用できます。

- カード番号: 4242424242424242
- 有効期限（月/年）: 本日より未来の年月（12/30など）
- CVC: 任意の３桁の数字

![](https://storage.googleapis.com/zenn-user-upload/b4fe63bfd030-20221005.png)

カード情報を入力し、申し込みを完了させましょう。

![](https://storage.googleapis.com/zenn-user-upload/3a7d2202566b-20221005.png)

申し込み後、StepFunctionsの管理画面にアクセスすると、ワークフローが開始していることが確認できます。

![](https://storage.googleapis.com/zenn-user-upload/7dd48c9787fa-20221004.png)


EventBridgeの管理画面では、「イベントを、ターゲットに送れたか否か」などの記録が確認できますので、デバッグに使用しましょう。

![](https://storage.googleapis.com/zenn-user-upload/e8c070c14359-20221005.png)
## [Advanced] 実運用を目指すための、チャレンジ項目

ワークショップの手順では、以下のケースがサポートできていません。

- プランによって、異なるCloudFormationテンプレートを起動させたい
- ホスティングプラン以外のサブスクリプションも作成・管理したい
- コンビニ決済や銀行振込など、「その場で決済が完了しない決済方法」をサポートする場合

以下にアイディアやヒントを用意しましたので、ぜひどのように実現するかをチャレンジしてみてください。

- Stripeの料金メタデータにCFNのURLを指定し、StepFunctions側でそのURLを利用する
- EventBridgeのフィルターにて、特定の料金IDを持つイベントのみ受け付けるルールを作る
- StepFunctionsで、「決済・入金が完了したか否か」を確認するステップを追加する
- 入金がまだの場合、Wait / Choiceを使って、入金が完了するまで待機し、タイムアウトした場合は解約処理に移る