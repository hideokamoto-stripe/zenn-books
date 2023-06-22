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

オンラインでの商品販売やサービス提供を開始すると、「クレジットカードの不正利用」への対策が必要になります。

Stripeの公開しているレポートでは、「2020 年 3 月から 5 月にかけて、不正使用関連以外の理由コード 「( 商品が届かない」「商品に不満がある」など) による申請が発生した決済取引は、2019 年の 2 倍以上に増加した」と紹介されています。

https://stripe.com/ja-de/guides/state-of-online-fraud

不正に入手されたクレジットカードでの注文が行われると、「本来のクレジットカード所有者から、カード会社に代金の返金を求める申請」が行われます。そしてカード会社がカード所有者の申請を承認した場合、支払われたはずの代金は顧客に返金されます。その時点では、すでに商品を発送している可能性が高いため、商品の原価・発送料などの損失が発生します。

Stripeでは、3Dセキュアによる本人確認認証や、Stripe Radarを利用した不正利用への対策を行うことができます。

https://stripe.com/ja-us/radar/chargeback-protection

ここでは、3Dセキュアの設定を変更して、不正利用のリスク削減を目指しましょう。

### 3Dセキュア認証のリクエスト閾値を変更する

3Dセキュアによる認証は、「カード会社が必要と判断した場合」だけでなく「認証を推奨する場合」や「すべての3Dセキュア認証に対応したクレジットカード」を対象にできます。

Stripeダッシュボードん[支払い]から[不正使用とリスク]を開きましょう。
![](https://storage.googleapis.com/zenn-user-upload/be51de4b06aa-20230526.png)

[ルール]タブに移動後、ページをスクロールして、[認証ルール]セクションに移動しましょう。

![](https://storage.googleapis.com/zenn-user-upload/8237c4a1953a-20230526.png)

ここでは、「3Dセキュア認証を要求する状態」の設定を、３段階から選べます。

:::message
**４段階目の設定**
「すべて無効化する」ことで、3Dセキュアの認証を行わないことも可能です。
が、不正利用のリスクが増加するため、お勧めしません。
:::

「認証が必須なカード」に加えて、「推奨する」とされたカードにも認証をリクエストしてみましょう。

「3Dセキュアが推奨されるカードの場合」の右側にある[...]ボタンをクリックして、[有効化]をクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/b3644739bc5a-20230526.png)

過去の決済データから、対象となる決済金額が算出されます。
![](https://storage.googleapis.com/zenn-user-upload/2bebad462ccf-20230526.png)

変更の影響が大きくないかを確認後、[ルールを有効にする]ボタンをクリックします。
![](https://storage.googleapis.com/zenn-user-upload/486862b4f081-20230526.png)

これで閾値の変更が完了しました。

### 3Dセキュア決済の分析も、ダッシュボードにて

3Dセキュアがどれくらいリクエストされて、成功・失敗やカゴ落ちなどがどれくらい起きたかを見ることができます。

ダッシュボードの[レポート]から[支払い認証レポート]を確認しましょう。

![](https://storage.googleapis.com/zenn-user-upload/76410a2245e5-20230525.png)

https://dashboard.stripe.com/authentication?startDate=2023-02-21&endDate=2023-05-22

ただし、このレポートはデータの反映までに最大24時間の遅延が発生します。

リアルタイムの結果については表示されませんので、ご注意ください。

### Radar for Teams(有料オプション)でより詳細なルール設定

有料オプションのRadar for Teamsを有効化すると、より詳細なルール設定も可能です。

https://stripe.com/ja-us/radar/fraud-teams

テスト環境では無料で試すことができますので、こちらも少し触ってみましょう。

[支払い]タブから[不正使用とリスク]を開きます。
![](https://storage.googleapis.com/zenn-user-upload/c5e04cf46e2b-20230525.png)

[ルール]タブに移動し、[+ルールを追加]ボタンをクリックしましょう。
![](https://storage.googleapis.com/zenn-user-upload/a4a9cef6bca2-20230525.png)

今回は、[3DSリクエストのルー ルを追加] をクリックします。
![](https://storage.googleapis.com/zenn-user-upload/1bd587da67c8-20230525.png)

高額商品の不正利用を回避するため、「指定金額以上の決済は、3Dセキュア認証を必須にする」条件を設定してみましょう。

ルールの入力欄に、「:amount_in_jpy: > 1000」と入力します。
![](https://storage.googleapis.com/zenn-user-upload/db8b71fedf06-20230525.png)

下部にある[ルールをテスト]ボタンをクリックすると、ルールのプレビュー画面が開きます。

[ルールを追加]をクリックして、ルールを確定させましょう。
![](https://storage.googleapis.com/zenn-user-upload/7ba160cf412d-20230525.png)

この状態で、「1,000円以上の商品を注文する支払いリンクURL」にアクセスし、クレジットカード決済を試しましょう。

すると、決済時にモーダル画面が一時的に開きます。これは3Dセキュア認証をテストモードで実施しているための挙動です。
![](https://storage.googleapis.com/zenn-user-upload/3b71d7fb5793-20230525.png)

拒否された時の挙動などは、テスト用に用意されたカード番号で確認できます。

https://stripe.com/docs/testing#regulatory-cards

## Checkpoint

- クロスセルなどの、客単価アップが１クリックで可能
- 不正利用対策で、3Dセキュア認証を無料で使える
- 有料のRadar for Teamsを使えば、決済額やIPアドレスなどを使った設定も可能