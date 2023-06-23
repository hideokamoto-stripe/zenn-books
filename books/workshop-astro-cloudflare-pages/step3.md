---
title: "サブスクリプション商品を販売する方法"
---

アプリの準備ができましたので、さっそくサブスクリプション申し込みフォームを用意しましょう。

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

## 支払いリンクでサブスクリプション申し込みを受け付ける方法

Stripeでは、「Payment Links」を利用して1回きりの注文もサブスクリプションの申し込みもコードを書かずに用意することができます。

### 支払いリンク作成画面に移動する

ダッシュボードを表示した状態で、 [c]キー -> [l]キーの順にキーボードを入力します。

![](https://storage.googleapis.com/zenn-user-upload/9058b6a5d7ce-20220419.png)


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

ここでは[新しい商品を追加]をクリックして、[商品登録画面]を開きます。

![](https://storage.googleapis.com/zenn-user-upload/f78096b891db-20230525.png)

### 商品とサブスクリプション料金プランを登録しよう

商品情報とサブスクリプションの料金プランを登録しましょう。

#### 商品情報を設定する

デモ用の商品として、以下のテキストと画像を入力・アップロードしましょう。

|商品名|画像（フリー素材DLページ）|説明文|
|:--|:--|:--|
|コーヒー豆定期お届け便|https://www.pakutaso.com/20170200038post-10294.html|厳選したコーヒー豆を、定期的にお届けします。|

![](https://storage.googleapis.com/zenn-user-upload/91990cbb468b-20230526.png)

|商品名|画像（フリー素材DLページ）|説明文|料金体系モデル|価格|通貨|種類|
|:--|:--|:--|:--|:--|:--|:--|
|焙煎済みコーヒー豆|https://www.pakutaso.com/20170200038post-10294.html|焙煎済みのコーヒー豆です。|標準の料金体系|1000|JPY|[一括]|

#### 料金プランを追加しよう

続いて、サブスクリプションの料金プランを追加します。

モーダル画面下部の、[価格]に移動しましょう。
![](https://storage.googleapis.com/zenn-user-upload/52669da529aa-20230526.png)

[継続]ボタンを選択すると、とサブスクリプションの料金プランを設定する画面が開きます。

![](https://storage.googleapis.com/zenn-user-upload/456dad3e8e99-20230526.png)

料金データを次のように入力しましょう。

|価格|通貨|請求期間|
|:--|:--|:--|
|1500|JPY|月次|



最後に[商品を追加]ボタンをクリックします。
![](https://storage.googleapis.com/zenn-user-upload/a448b746bb3e-20230526.png)

作成したプランが、支払いリンクに登録されました。

![](https://storage.googleapis.com/zenn-user-upload/d94bf024cd48-20230526.png)
[リンクを作成]ボタンをクリックすると、サブスクリプション申し込みの支払いリンクURLが作成されます。
![](https://storage.googleapis.com/zenn-user-upload/1a8ded1e43ea-20230526.png)

## Astroサイトに、「サブスクリプション申し込みボタン」を埋め込もう

作成した支払いリンクへの導線を、アプリに組み込みましょう。

もっとも簡単な方法は、`a`タグで取得した支払いリンクURLへのテキストリンクを作ることです。

```diff html:src/pages/index.astro
---
import Layout from '../layouts/Layout.astro';
import Card from '../components/Card.astro';
---

<Layout title="Welcome to Astro.">
	<main>
+		<a href="<取得した支払いリンクURL>">サブスクリプションに申し込む</a>
	</main>
</Layout>
```

ただしこの方法では、せっかくStripeに登録した商品・料金情報を活かすことができません。

そこで[購入ボタン（Buy Button）]を支払いリンクページから生成しましょう。

### 購入ボタン（Buy Button）を作る方法

[購入ボタン]は、支払いリンクの詳細ページ（URLを取得した画面）から生成できます。

[QRコード]ボタンの右隣にある[購入ボタン]ボタンをクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/393cde04fbe2-20230623.png)

画面右側に、コードスニペットとボタンのプレビューが表示されます。

![](https://storage.googleapis.com/zenn-user-upload/0c333f347e92-20230623.png)

表示されたスニペットをコピーして、`src/pages/index.astro`に貼り付けましょう。

```diff html:src/pages/index.astro
---
import Layout from '../layouts/Layout.astro';
import Card from '../components/Card.astro';
---

<Layout title="Welcome to Astro.">
	<main>
-		<a href="<取得した支払いリンクURL>">サブスクリプションに申し込む</a>
+		<script async
+		  src="https://js.stripe.com/v3/buy-button.js">
+		</script>
+		
+		<stripe-buy-button
+		  buy-button-id="buy_btn_xxxxx"
+		  publishable-key="pk_test_xxxxxx"
+		>
+		</stripe-buy-button>
	</main>
</Layout>
```

ボタンのみ埋め込みしたい場合は、[設定]を[カード]から[ボタン]に変更します。

![](https://storage.googleapis.com/zenn-user-upload/a1c20cd8cc88-20230623.png)


また、配色やボタンテキストのカスタマイズも行えます。

![](https://storage.googleapis.com/zenn-user-upload/6c1a84959f2d-20230623.png)

変更の保存と反映は、パネル下部にある[変更を保存してコードをコピーする]ボタンをクリックします。

コードスニペットが変わることはないため、埋め込み済みの購入ボタンにも変更を反映できます。

## [Advanced] より便利にPayment Linksを使う Tips

### Tips1: 　「初回契約手数料」などの、「１回きりの請求」を一緒に行う方法

ビジネスモデルや事務手続の関係で、初回の契約時のみ追加の請求を行いたいことがあります。

その場合は、支払いリンクの商品に「サブスクリプションの料金」と「１回きりの商品」両方を登録しましょう。

![](https://storage.googleapis.com/zenn-user-upload/6def78011d07-20230526.png)

決済ページでも、「初回のみXX円、次回からはYY円」と表示されます。

![](https://storage.googleapis.com/zenn-user-upload/27ac0a80572b-20230526.png)

### Tips2: 「トライアル期間」を設定しよう

食品などの物理的なアイテムの定期販売では難しいですが、個人レッスンやオンラインサービスでは「まずは7日間お試しを」などのオファーを出すことができます。

支払いリンクを作成・編集する際に、[オプション]で[無料トライアルを含める]をオンにしましょう。
![](https://storage.googleapis.com/zenn-user-upload/9c75aa0aaa79-20230526.png)

日数を入力できますので、[7]や[14]などを入れましょう。
![](https://storage.googleapis.com/zenn-user-upload/ea2282bf1bd3-20230526.png)

リンクを更新すると、決済画面に[無料期間]の表示が追加されます。

![](https://storage.googleapis.com/zenn-user-upload/c494df7d6e8e-20230526.png)



## Checkpoint

- Payment Linksでは、サブスクリプション契約も提供できる
- 購入ボタンを使えば、登録した写真や料金情報の埋め込みもできる
- 配送料金など、一部利用できない機能がある点に注意