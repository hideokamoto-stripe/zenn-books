---
title: "Payment Links(支払いリンク)をとりあえず作ってみよう"
---

「Payment Linksを使って支払いリンクを作成すること」自体は、5分も必要ありません。

「習うより慣れよ」とも言いますので、ここではいきなりテスト環境で支払いリンクを作成してみましょう。

## Stripeダッシュボードに ログイン

まずはStripeダッシュボードにログインしましょう。

https://dashboard.stripe.com/

### Stripeアカウントは、メールアドレスだけで作成可能

データを登録するためには、Stripeアカウントが必要です。
まだアカウントを持っていない方は、以下のURLからアカウントを作成してください。

https://dashboard.stripe.com/register


![](https://storage.googleapis.com/zenn-user-upload/939f923c3dfc-20220419.png)


### 新しいテスト用アカウントを作成しよう

まずは、今回のテストアプリ用のストアアカウントを作成します。

Stripeでは、ストアやサービス毎に複数のアカウントを作成・管理することが可能です。
アカウントを分けることで、APIの動作確認やワークショップ中の事故で顧客データや請求内容に予期せぬ変更が入るリスクなどを回避できます。

ページ左上のアイコンをクリックし、[新規アカウント]をクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/3fcedd882d34-20220419.png)

アカウントでは、ビジネス・企業名とどの国で業務を行なっているかの２点を設定します。

今回はデモアプリですので、以下のように設定しましょう。

- アカウント名: demo.特産品.com
- 業務をおこなっている国: 日本

![](https://storage.googleapis.com/zenn-user-upload/7f654a649b6b-20220419.png)

設定が完了すると、アカウントのTOPページに移動します。

![](https://storage.googleapis.com/zenn-user-upload/9058b6a5d7ce-20220419.png)

#### Tips: 本番環境での利用について

アカウント作成すると、テスト環境が用意されます。

テスト環境では、「Stripeが用意するテストカード番号以外の入力を拒否する」「請求書などのメールを送信できるアドレスが制限されている」などの制限があります。

実際に顧客の決済を受け付けるには、なるべく早い段階で本番環境の利用申請を行うようにしましょう。

![](https://storage.googleapis.com/zenn-user-upload/b0d0b1a434d3-20220419.png)

なお、「サービス内容の詳細などの情報が不十分だと判断された場合」や「[禁止/制限付き業種](https://stripe.com/ja-it/legal/restricted-businesses)に該当すると判断された場合」、アカウントが停止される可能性があります。禁止・制限付き業種に該当しないかの事前に確認や、できるだけ詳細なビジネス情報の入力をお勧めします。

## Payment Linksで「支払いリンク」を作成しよう

Stripeアカウントの準備ができましたので、支払いリンクを作成しましょう。

### 支払いリンク作成画面に移動する
ダッシュボードを表示した状態で、 [c]キー -> [l]キーの順にキーボードを入力します。

![](https://storage.googleapis.com/zenn-user-upload/9058b6a5d7ce-20220419.png)

すると支払いリンクを作成する画面が開きます。

![](https://storage.googleapis.com/zenn-user-upload/6077c0b5f263-20230525.png)

このように、Stripeダッシュボードでは、操作を簡単にするための[キーボードショートカット]が複数用意されています。

![](https://storage.googleapis.com/zenn-user-upload/85f46fa1c303-20230526.png)

#### その他の支払いリンク作成画面の開き方
キーボードショートカット以外にも、いくつかのアクセス方法が用意されています。

##### ヘッダーメニューから移動する
ページ上部の[作成]ボタンをクリックし、[支払いリンク]を選択しましょう。
![](https://storage.googleapis.com/zenn-user-upload/69bed5e60a70-20230525.png)

##### URLで直接移動する
https://paymentlinks.new をブックマークすることで、URLから直接移動することも可能です。


### 請求するための商品を登録しよう

支払いリンクでは、注文する商品や申し込みするサブスクリプションプランの登録が必要です。

![](https://storage.googleapis.com/zenn-user-upload/6077c0b5f263-20230525.png)

画面左側の設定画面にある、[商品]セクションの検索フォームをクリックしましょう。

[新しい商品を追加] ボタンと、すでに登録されている商品のリストが表示されます。

![](https://storage.googleapis.com/zenn-user-upload/e1637ddc7ebb-20230525.png)

ここでは[新しい商品を追加]をクリックします。

すると[商品登録画面]が開きます。

![](https://storage.googleapis.com/zenn-user-upload/f78096b891db-20230525.png)

ここでは、「コーヒー豆をオンライン販売するお店」を想定して、次の商品を登録しましょう。

|商品名|画像（フリー素材DLページ）|説明文|料金体系モデル|価格|通貨|種類|
|:--|:--|:--|:--|:--|:--|:--|
|焙煎済みコーヒー豆|https://www.pakutaso.com/20170200038post-10294.html|焙煎済みのコーヒー豆です。|標準の料金体系|1000|JPY|[一括]|

商品情報をそれぞれの入力欄に追加できたら、 [商品を追加]を クリックします。
![](https://storage.googleapis.com/zenn-user-upload/e4c32e4f3536-20230525.png)

これで編集中の支払いリンクに 商品が追加されました。
![](https://storage.googleapis.com/zenn-user-upload/d14f28553922-20230525.png)

あとは画面右上にある[リンクを作成]を クリックすると・・・
![](https://storage.googleapis.com/zenn-user-upload/f9a16f0361fa-20230525.png)

これだけで注文用のURLが発行できました。

![](https://storage.googleapis.com/zenn-user-upload/e45aca6b0e93-20230525.png)

表示されているURLをコピーしたり、QRコードをダウンロードして共有することで、顧客からの注文・決済を受け付けることができます。

## 作成した支払いリンクを、顧客に共有・サイトに掲載しよう

作成した支払いリンクを使って、さまざまな方法で決済の受付やサブスクリプションの申し込みが行えます。

ここでは「URLの共有」「QRコードの発行」「購入ボタンの埋め込み」の3つを紹介します。

### 支払いリンクURLを共有する

もっとも簡単な方法は、URLを取得することです。

支払いリンク管理画面に、[コピー]ボタンがありますので、クリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/7f4ba71e8e68-20230525.png)


これでURLをコピーしました。

あとはWordPressやSTUDIOなどのサイトビルダーやCMSにURLをリンクとして設定するだけで、組み込みが完了します。

![](https://storage.googleapis.com/zenn-user-upload/79116c2e516f-20230526.png)


### QRコードを発行する方法
ポップアップストアや移動販売などでは、URLではなくQRコードを顧客のスマートフォンに読み込んで決済してもらうこともできます。

支払いリンク管理画面にある、[QR コード]ボタンをクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/7f4ba71e8e68-20230525.png)

QRコードが表示されました。

[画像をダウンロード]でデータをダウンロードし、直接印刷やカメラロールから顧客に共有することができます。

また、[クリップボードにコピー]ボタンを利用すると、データをコピーできます。CanvaやFigmaなどのデザインツールで編集したい場合は、こちらを使いましょう。

![](https://storage.googleapis.com/zenn-user-upload/3372a5f246e6-20230525.png)

### 購入ボタンのコードスニペットを埋め込む

2023年から、支払いリンクをボタンまたはカード形式でサイトに埋め込むことができるようになりました。

https://stripe.com/docs/payment-links/buy-button


支払いリンク管理画面にある、[購入ボタン]ボタンをクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/7f4ba71e8e68-20230525.png)

クリックすると、サイトやアプリに埋め込む購入ボタンの編集画面が開きます。

![](https://storage.googleapis.com/zenn-user-upload/a7a7926ab354-20230526.png)

「商品情報をStripeとCMSで二重管理したくない」場合や、「LPやブログで手軽に埋め込みしたい」場合には、商品名や画像・価格が表示される[カード]を選びましょう。

![](https://storage.googleapis.com/zenn-user-upload/4a8464f56a5b-20230526.png)

すでにあるレイアウトに埋め込みたい場合など、「あくまでボタンだけが欲しい場合」には[ボタン]を選択します。

![](https://storage.googleapis.com/zenn-user-upload/9a8fb3db4743-20230526.png)

背景やボタンの色なども変更できます。

![](https://storage.googleapis.com/zenn-user-upload/ed43352326e6-20230526.png)

このように、URL・QRコード・コードスニペットの３種類を必要な場面や状況に併せて使い分けることができます。

## Checkpoint

- Stripeアカウントは、**テスト目的**ならメールアドレスだけあれば作れる
- Payment Linksで支払いリンクを作るショートカットは https://paymentlinks.new
- QRコードやURL・購入ボタンで、簡単に決済フォームを顧客にシェアしよう

次のステップでは、作成した支払いフォームのカスタマイズを行います。