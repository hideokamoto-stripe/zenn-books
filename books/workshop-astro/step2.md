---
title: "リンクまたは埋め込みコードで決済機能を実装する"
---

もっとも簡単な方法は、Stripe上で発行した決済リンクや埋め込みコードを使うことです。

### Payment Linksで発行したURLを利用する

StripeのPayment Linksを使うことで、決済ページへのURLを発行できます。

![スクリーンショット 2023-02-07 12.33.53.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2366300/01e28d58-4a37-acff-6a48-efd169a26722.png)

発行したURLまたはQRコードを、サイトのリンクや画像として埋め込みましょう。

![スクリーンショット 2023-02-07 12.34.25.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2366300/f9648362-7a4e-a72a-01f1-caa160825bf4.png)

```js
---
import Layout from '../layouts/Layout.astro';
import Card from '../components/Card.astro';

---

<Layout title="Welcome to Astro.">
	<main>
		<ul role="list" class="link-card-grid">
			<Card
				href="https://buy.stripe.com/test_xxxxxx"
				title="コーヒー豆を注文する"
				body="おすすめのコーヒー豆をお届けします。（税込1,500円）"
			/>
```

Astro側で注文ボタンなどを用意するだけで、決済やサブスクリプションの申し込みフローを実装できます。

![スクリーンショット 2023-02-07 12.36.41.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2366300/bd32c45b-3d48-b5fb-5e92-08433194492e.png)

### 料金表を作成して埋め込む

プランによって提供する機能が変わる場合などでは、料金表を作成して埋め込むこともできます。

![スクリーンショット 2023-02-07 12.38.20.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2366300/5c10bf08-9c74-4566-792d-d1841e524e0b.png)

発行されたコードを、そのままAstroのコードに追加しましょう。

```js
---
import Layout from '../layouts/Layout.astro';
import Card from '../components/Card.astro';
---

<Layout title="Welcome to Astro.">
	<main>
		<h1>Welcome to <span class="text-gradient">Astro</span></h1>
		<script async src="https://js.stripe.com/v3/pricing-table.js"></script>
		<stripe-pricing-table pricing-table-id="prctbl_1xxxxxxx"
		publishable-key="pk_test_xxxxxx">
		</stripe-pricing-table>
		<ul role="list" class="link-card-grid">
```

これだけで料金表と決済フォームへの遷移が実装できます。

![スクリーンショット 2023-02-07 12.39.17.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2366300/7616de2c-2c9b-3eca-f1ba-8ca7ce4cc2b6.png)


https://stripe.com/docs/payments/checkout/pricing-table

## TODO

Buy Buttonも触れる