---
title: "Stripeで、「料金表」を作成して埋め込もう"
---

登録したサブスクリプション商品を、アプリから申し込めるようにしましょう。

まずは、料金表のJSスニペットを埋め込む方法を試します。

## Stripeダッシュボードで、料金表を作成しよう

2022年のアップデートで、「料金表」をサイトに埋め込むコードスニペットが作れるようになりました。

![](https://storage.googleapis.com/zenn-user-upload/60ec6ac497b7-20230424.png)

ダッシュボードの[商品 > 料金表]に移動しましょう。

![](https://storage.googleapis.com/zenn-user-upload/b2689ef63f3b-20230424.png)

[料金表を作成する]をクリックすると、料金表の作成画面が開きます。

![](https://storage.googleapis.com/zenn-user-upload/e1c59611b83b-20230424.png)

### 商品を料金表に追加する
[テストの商品を見つけるまたは追加する]のフォームをクリックして、商品を追加しましょう。

![](https://storage.googleapis.com/zenn-user-upload/efb21876529c-20230424.png)

追加すると、右側のプレビュー画面にも商品が表示されます。

![](https://storage.googleapis.com/zenn-user-upload/23587d3a055b-20230424.png)

[別の商品を追加]をクリックして、複数の商品を登録しましょう。

![](https://storage.googleapis.com/zenn-user-upload/b654a53d8be3-20230424.png)

:::details [Appendix]「最上位プランはお問い合わせください」を実現する方法
料金表に決済画面ではなく、任意のフォームやページへのリンクを追加できます。
[カスタムの行動換気ボタンで製品を追加]をクリックして、リンクURLを設定しましょう。

![](https://storage.googleapis.com/zenn-user-upload/6dce0a1a398f-20230424.png)
:::

### 料金表の見た目をカスタマイズしよう

料金表は、Payment Links同様いくつかのカスタマイズ機能が用意されています。

[表示設定]から、言語やボタン・背景の配色などを変更しましょう。

#### 言語を変更する

まずは[言語]を選択しましょう。

![](https://storage.googleapis.com/zenn-user-upload/0ae6af449579-20230425.png)

テキストが入力できるセレクトボックスが表示されます。

[日本語]を入力して、日本語を設定しましょう。

![](https://storage.googleapis.com/zenn-user-upload/b195803be9c1-20230425.png)

[プレビュー]に表示されている料金表のボタンテキストなどが、日本語に変わりました。

![](https://storage.googleapis.com/zenn-user-upload/ef67f858b826-20230425.png)

#### おすすめ商品を強調表示する

複数の料金を表示する場合、お勧めしたい商品を強調表示できます。

[商品を強調表示]をオンにします。
![](https://storage.googleapis.com/zenn-user-upload/e1881d9fad18-20230425.png)

[お勧めしたい商品]と、[表示するラベル名]を選べます。

![](https://storage.googleapis.com/zenn-user-upload/23c7c65ebf24-20230425.png)

右側のプレビュー画面で、選択した商品が強調表示されていることが確認できます。

![](https://storage.googleapis.com/zenn-user-upload/c9534f683e95-20230425.png)

ラベルは、2023/4時点で[最も人気][最もお得][おすすめ]の３つから選べます。
![](https://storage.googleapis.com/zenn-user-upload/59a384114833-20230425.png)

## 決済方法の設定を行おう

料金表に表示したい商品や見た目の設定が終われば、[続行]ボタンをクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/bae7f5819c8b-20230425.png)

料金表のボタンをクリックした後に表示される、決済フォームの設定画面に移動します。

![](https://storage.googleapis.com/zenn-user-upload/96a690763c08-20230425.png)

ここでは、2023/04時点で次のようなカスタマイズができます。

- プロモーションコード（クーポン）の適用
- 納税者番号（日本では法人番号）の入力
- 注文数量の変更
- 請求先住所・発送先住所フォームの追加
- 電話番号フォームの追加
- カスタムフィールドの追加（２つまで）
- [Stripe Tax利用時] 消費税の自動計算

自由に設定して構いません。

もしどのように設定すれば良いか悩まれた場合は、下のスクリーンショットを参考に設定しましょう。

![](https://storage.googleapis.com/zenn-user-upload/d523e1ad73ca-20230425.png)

また、支払い完了後のThanksページの設定も行えます。

![](https://storage.googleapis.com/zenn-user-upload/c3f2507155de-20230425.png)

今回は[確認ページを表示する]を選択しましょう。

[続行]ボタンをクリックして、次のステップに移りましょう。

![](https://storage.googleapis.com/zenn-user-upload/ca7a437411f5-20230425.png)

## 顧客マイページの設定を行おう

サブスクリプションビジネスでは、クレジットカード情報の更新フォームやプラン変更・キャンセル画面、請求履歴を表示するページなどの機能が欠かせません。

次のステップでは、このようなサブスクリプション申し込み情報を管理する[カスタマーポータル]を作成します。

![](https://storage.googleapis.com/zenn-user-upload/839cb849c930-20230425.png)

このステップでは、「顧客が他のプランに変更できるようにするか」と、「どの商品・料金への変更を許可するか」を設定できます。

デフォルトでは、料金表に追加した商品・料金が選択されています。

![](https://storage.googleapis.com/zenn-user-upload/abd62bfe78b1-20230425.png)

今回のワークショップでは、設定を変更せずに[完了]をクリックしましょう。

## コードスニペットを取得しよう

料金表の作成に成功すると、サイトに埋め込むためのコードスニペットが生成されます。

![](https://storage.googleapis.com/zenn-user-upload/dbc44524c7fb-20230425.png)

表示されたスニペットをコピーして、`src/pages/index.astro`に貼り付けましょう。

```diff html:src/pages/index.astro
---
import Layout from '../layouts/Layout.astro';
import Card from '../components/Card.astro';
---

<Layout title="Welcome to Astro.">
	<main>
+		<script async src="https://js.stripe.com/v3/pricing-table.js"></script>
+		<stripe-pricing-table
+			pricing-table-id="prctbl_xxxx"
+			publishable-key="pk_test_xxx"
+		>
+		</stripe-pricing-table>
	</main>
</Layout>
```

`npm run dev`でアプリを起動し、http://localhost:3000 にアクセスしましょう。

プレビューに表示されていた料金表が表示されていれば、埋め込み完了です。

![](https://storage.googleapis.com/zenn-user-upload/3505de873400-20230425.png)

## 顧客向けの請求マイページURLを設定しよう

続いて、サブスクリプション申し込み済みのユーザー向けに請求マイページを用意しましょう。

Stripeダッシュボードの、料金表コードスニペットを取得したページに再度アクセスします。

![](https://storage.googleapis.com/zenn-user-upload/72709486aa04-20230425.png)

ページを下にスクロールして、[カスタマーポータル]セクションに移動しましょう。

`https://billing.stripe.com`から始まるURLが表示されていますので、コピーします。

![](https://storage.googleapis.com/zenn-user-upload/65350ff7d7a6-20230425.png)

Astroのアプリで、このURLに移動するリンクを追加すれば、マイページの設定は完了です。

```html:src/pages/index.astro
		</stripe-pricing-table>
+		<a
+			href="https://billing.stripe.com/p/login/test_xxxx"
+			target="_blank"
+			rel="noreferrer noopener"
+		>マイページ (メール認証)</a>
	</main>
</Layout>
```

追加したリンクをクリックすると、顧客マイページへのログイン画面が開きます。

![](https://storage.googleapis.com/zenn-user-upload/4dc5e33d88ea-20230425.png)

テスト環境では、**「Stripeダッシュボードにログインできるメールアドレス」**かつ**Stripeアカウントに、Customerデータが作成されている**場合のみ利用できます。

料金表を使ったサブスクリプション申し込みのテストを行う際、１度は **「Stripeダッシュボードにログインできるメールアドレス」** で申し込みを行いましょう。

:::message
テスト環境では、以下のカード情報を使いましょう。

- カード番号: `4242424242424242`
- 有効期限: 現在より未来の年月ならなんでもOK (`01`/`40`)
- CVC: ３桁の数字ならなんでもOK (`333`)
:::

:::details [Appendix]カスタマーポータルで顧客が操作できる内容を変更する
[コピー]ボタン横にある歯車ボタンをクリックすると、カスタマーポータルの設定画面が開きます。

ここでは、カスタマーポータルにでユーザーができる操作を指定できます。

![](https://storage.googleapis.com/zenn-user-upload/bd700feb63c3-20230425.png)

「キャンセル操作はさせてもよいが、プラン変更はさせたくない」や、
「請求履歴や領収書PDFのダウンロード・決済情報の更新だけできるようにしたい」など、
要件に応じた設定のカスタマイズを行いましょう。

:::


## テスト環境と本番環境を切り替えれるようにする

料金表をサービスに組み込むには、本番環境とテスト環境で読み込む料金表を変更できる必要があります。

そこで環境変数を利用して、料金表を切り替えれるようにしましょう。

料金表のコードスニペットから、`pricing-table-id`と`publishable-key`の２つをコピーしましょう。

`.env`ファイルを`package.json`などがあるディレクトリに作成し、次のように設定します。

```env:.env
PUBLIC_STRIPE_PUBLISHABLE_API_KEY=<publishable-keyの値>
PUBLIC_STRIPE_PRICING_TABLE_ID=<pricing-table-idの値>
```

また、`src/pages/index.astro`に追加している、「カスタマーポータルのURL」も設定しましょう。

```diff:.env
PUBLIC_STRIPE_PUBLISHABLE_API_KEY=<publishable-keyの値>
PUBLIC_STRIPE_PRICING_TABLE_ID=<pricing-table-idの値>
+PUBLIC_STIPE_CUSTOMER_PORTAL_URL=<カスタマーポータルのURL>
```

続いて、`src/pages/index.astro`で環境変数を読み込むようにします。

```diff html:src/pages/index.astro
---
import Layout from '../layouts/Layout.astro';
import Card from '../components/Card.astro';
+const {
+	PUBLIC_STRIPE_PUBLISHABLE_API_KEY,
+	PUBLIC_STRIPE_PRICING_TABLE_ID,
+	PUBLIC_STIPE_CUSTOMER_PORTAL_URL
+} = import.meta.env;

const now = new Date()
---
<Layout title="Welcome to Astro.">
	<main>
		<script async src="https://js.stripe.com/v3/pricing-table.js"></script>
		<stripe-pricing-table
-			pricing-table-id="prctbl_xxxx"
-			publishable-key="pk_test_xxx"
+			pricing-table-id={PUBLIC_STRIPE_PRICING_TABLE_ID}
+			publishable-key={PUBLIC_STRIPE_PUBLISHABLE_API_KEY}
		>
		</stripe-pricing-table>
		<a
-			href="https://billing.stripe.com/p/login/test_xxxx"
-			href={PUBLIC_STIPE_CUSTOMER_PORTAL_URL}
			target="_blank"
			rel="noreferrer noopener"
		>マイページ (メール認証)</a>
	</main>
</Layout>

```

この状態で、`npm run dev`を再度実行しましょう。

料金表の表示が変わらなければ、切り替え成功です。

## Cloudflare Pagesに環境変数を保存する

作成した料金表を本番環境で動作させるため、Cloudflare側でも設定を行いましょう。

Cloudflareダッシュボード（ https://dash.cloudflare.com/ ）にログインしましょう。

![](https://storage.googleapis.com/zenn-user-upload/389057601dd1-20230425.png)

ワークショップで作成したPagesプロジェクトを開きます。

![](https://storage.googleapis.com/zenn-user-upload/b045c9785029-20230425.png)

[Settings]タブの[Environment Variables]に移動します。

![](https://storage.googleapis.com/zenn-user-upload/20ad607c75bc-20230425.png)

[Production]の[Add variables]ボタンをクリックすると、環境変数を設定する画面が開きます。

`PUBLIC_STRIPE_PUBLISHABLE_API_KEY`と`PUBLIC_STRIPE_PRICING_TABLE_ID`、そして`PUBLIC_STIPE_CUSTOMER_PORTAL_URL`を追加しましょう。

![](https://storage.googleapis.com/zenn-user-upload/ffa2b7d7d421-20230425.png)

:::message
実運用では、`Production`にはStripeの本番環境のデータを登録します。
また、プレビュー環境も用意したい場合は、`Preview`側にStripeのテスト環境のデータを保存しましょう。
:::

### 変更を、Cloudflareにデプロイする

Cloudflare Pages上でStripeの料金表を表示する準備ができました。

Astroでのビルドと、Wranglerを使ったデプロイを行いましょう。

```bash
% npm run build
% wrangler pages publish ./dist
```

ビルドとデプロイ、そして環境変数の設定が完了していれば、Cloudflare Pages側のURLでも料金表が表示されます。

![](https://storage.googleapis.com/zenn-user-upload/dfb20682cb80-20230425.png)

## おさらい

- Stripeの料金表で、簡単にSaaSの料金表を作成・埋め込みできる
- 実運用時には、料金表IDの公開可能APIキーを環境変数から読み込ませよう
- Cloudflare Pagesの環境変数は、ダッシュボード上から登録しよう

次のステップでは、サブスクリプションの申し込みを受け取るREST APIを作成します。