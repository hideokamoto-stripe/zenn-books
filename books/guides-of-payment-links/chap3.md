---
title: "売上を増やす提案や、不正利用対策など、収益を増やす施策を試してみよう"
---

Stripe Payment Linksでは、この他にも売上を増やしたり出費を抑えるための設定が可能です。

ここでは、クロスセルと3Dセキュアの設定を見てみましょう。

## クロスセルを提案してみよう

「ご一緒にポテトはいかがでしょうか？」を代表するように、商品注文時に追加の提案を行うのが「クロルセル」です。

Stripeでは、「１商品につき１つ」クロスセルの提案設定ができます。

ダッシュボードの[商品]ページにて [商品を追加]をクリックしましょう。
![](https://storage.googleapis.com/zenn-user-upload/b3044b11a789-20230525.png)

クロルセルで提案したい商品の情報を入力して保存します。
![](https://storage.googleapis.com/zenn-user-upload/9d89c108e8bb-20230525.png)

すでに作成、支払いリンクへの登録を済ませている商品の詳細ページに移動しましょう。
![](https://storage.googleapis.com/zenn-user-upload/20cf26d0777b-20230525.png)

ページをスクロールすると、[クロスセル]が表示されます。

ここで先ほど登録した商品を検索し、選択しましょう。

![](https://storage.googleapis.com/zenn-user-upload/0d43e1212eba-20230525.png)

このように、商品名が表示されていれば、登録完了です。
![](https://storage.googleapis.com/zenn-user-upload/95b13159d738-20230525.png)

支払いリンクURLにアクセスすると、登録した商品のレコメンドボタンが追加されていることがわかります。
![](https://storage.googleapis.com/zenn-user-upload/905e431f28c1-20230525.png)

## 不正請求対策をはじめよう

[支払い]タブから[不正使用とリスク]を開く
![](https://storage.googleapis.com/zenn-user-upload/c5e04cf46e2b-20230525.png)

[+ルールを追加]ボタンをクリック
![](https://storage.googleapis.com/zenn-user-upload/a4a9cef6bca2-20230525.png)

[3DSリクエストのルー ルを追加] をクリック
![](https://storage.googleapis.com/zenn-user-upload/1bd587da67c8-20230525.png)

「指定金額以上の決済は、3DS必須」にする :amount_in_jpy: > 10
![](https://storage.googleapis.com/zenn-user-upload/db8b71fedf06-20230525.png)

ルールのプレビュー （過去の決済データで照合）
[ルールを追加]を クリックして確定
![](https://storage.googleapis.com/zenn-user-upload/7ba160cf412d-20230525.png)

![](https://storage.googleapis.com/zenn-user-upload/76410a2245e5-20230525.png)

決済時に3DS2の 認証が入るように
![](https://storage.googleapis.com/zenn-user-upload/3b71d7fb5793-20230525.png)

パフォーマンスレポートは最大24時間遅延する
![](https://storage.googleapis.com/zenn-user-upload/76410a2245e5-20230525.png)

:::message
Radar for Teamsは有料
:::