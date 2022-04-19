---
title: "Stripeを利用して、商品・料金情報を登録しよう"
---

ここからは、商品や料金情報をStripeで管理する方法を紹介します。

## Stripeアカウントを用意しよう

データを登録するためには、Stripeアカウントが必要です。
まだアカウントを持っていない方は、以下のURLからアカウントを作成してください。

https://dashboard.stripe.com/register

![](https://storage.googleapis.com/zenn-user-upload/939f923c3dfc-20220419.png)

アカウントを作成後、ダッシュボードにログインしましょう。

https://dashboard.stripe.com/
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

## 商品・料金を登録しよう

それでは、さっそく商品を登録しましょう。

ダッシュボードのメニューから[商品]をクリックします。

![](https://storage.googleapis.com/zenn-user-upload/51ef7e37f6cb-20220419.png)
直接リンク: https://dashboard.stripe.com/test/products

[商品を追加]ボタンがありますので、クリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/bfb227ca5ca5-20220419.png)

商品の登録画面が開きます。

### 商品の基本情報を登録しよう

デモ用の商品として、以下のテキストと画像を入力・アップロードしましょう。

|商品名|画像（フリー素材DLページ）|説明文|
|:--|:--|:--|
|松坂牛ステーキ２枚セット|https://www.pakutaso.com/20210514144post-34975.html|松坂牛のステーキ２枚セットです。商品は冷凍してお届けします。|

![](https://storage.googleapis.com/zenn-user-upload/9d44ad1d05f8-20220419.png)

### 価格情報（料金）を登録して保存しよう

続いて商品の価格を登録しましょう。

ページをスクロールして[料金情報]フォームを表示し、以下のデータを入力します。

|料金体系モデル|価格|通貨|種類|
|:--|:--|:--|:--|
|標準の料金体系|15000|JPY|[一括]|

![](https://storage.googleapis.com/zenn-user-upload/efe0e1a23c0f-20220419.png)

入力が終われば、ページ右上の[商品を保存]をクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/39b9c04e8148-20220419.png)

商品ページに移動すれば、登録完了です。

![](https://storage.googleapis.com/zenn-user-upload/4324df994563-20220419.png)
### 「XつでY円」の「パッケージ価格」を設定する

Stripeではパッケージ単位での価格設定も可能です。

新しく以下の内容で商品・料金を登録しましょう。

**商品情報**
|商品名|画像（フリー素材DLページ）|説明文|
|:--|:--|:--|
|とうもろこし|https://www.pakutaso.com/201707582052-24.html|産地直送のとうもろこしです。|

**料金情報**
|料金体系モデル|価格|通貨|単位|種類|
|:--|:--|:--|:--|:--|
|パッケージ料金体系|500|JPY|3ユニットごと|[一括]|

![](https://storage.googleapis.com/zenn-user-upload/c4e9b26c3e5d-20220419.png)

### 複数の商品を追加しよう

テスト用のデータとして、いくつかのサンプルを以下に用意しました。
全て登録する必要はありませんが、多い方が通販サイトらしくなるかと思いますので、時間に余裕のある方はぜひ登録してみましょう。


|商品名|画像（フリー素材DLページ）|説明文|料金体系モデル|価格|通貨|種類|
|:--|:--|:--|:--|:--|:--|:--|
|季節の野菜セット|https://www.pakutaso.com/20211259357post-38199.html|季節に応じた野菜を厳選してお届けします|標準の料金体系|2000|JPY|[一括]|
|あまおう（小粒）|https://www.pakutaso.com/20190253059post-19776.html|小粒のあまおうを１パックにまとめました|標準の料金体系|700|JPY|[一括]|
|焙煎済みコーヒー豆|https://www.pakutaso.com/20170200038post-10294.html|焙煎済みのコーヒー豆です。|標準の料金体系|1000|JPY|[一括]|
|コーヒードリッパー（布）|https://www.pakutaso.com/20200631170post-28054.html|布製のコーヒードリッパーです。|標準の料金体系|100|JPY|[一括]|

### クロスセルを設定しよう

StripeのCheckoutでは、決済ページでクロスセルの提案が行えます。

上のリストから、「焙煎済みコーヒー豆」と「コーヒードリッパー（布）」を登録しましょう。

登録後、ダッシュボードの[商品一覧ページ](https://dashboard.stripe.com/test/products?active=true)で[焙煎済みコーヒー豆]を選択します。

![](https://storage.googleapis.com/zenn-user-upload/c272e313be5a-20220419.png)

商品詳細ページ中段に[クロスセル]セクションが見つかります。

![](https://storage.googleapis.com/zenn-user-upload/6bc6e6d6e856-20220419.png)

[商品を検索]をクリックし、[コーヒードリッパー]を選択しましょう。

![](https://storage.googleapis.com/zenn-user-upload/cbfa09054263-20220419.png)

これでクロスセルの準備は完了です。

![](https://storage.googleapis.com/zenn-user-upload/4b49d14bcebe-20220419.png)

変更または削除したい場合は、[x]アイコンをクリックしましょう。



## おさらい

- Stripeでは、１ビジネス(サービス/ストア)毎にアカウントを作れる
- 商品名・画像・価格などがダッシュボードから登録できる
- クロスセルや段階的な値引きなどの複雑な設定も可能

次のステップでは、登録した商品データをNext.jsアプリで取得・表示します。

## 「早く終わった」という方のための、もう１ステップ

Stripeでは、カード決済以外に銀行振込での請求書送付にも対応しています。

以下の記事を参考に、Stripeでの請求業務を試してみましょう。

https://qiita.com/hideokamoto/items/31c4ffd0d90bff654254