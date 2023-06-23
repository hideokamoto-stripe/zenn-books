---
title: "サブスクリプションの請求マイページを作ろう"
---


サブスクリプションビジネスを提供する場合、契約後に顧客からさまざまな問い合わせや依頼が届きます。

- 「カードを紛失したので、新しいカードに変更したい」のような決済手段の更新に関するもの
- 「領収書が必要なので、過去3ヶ月分のPDFをください」など、何かしらの経理的な事情があると思われるもの
- 「解約したい・3ヶ月休みたい」などの契約に関するもの
- etc..

多種多様な問い合わせに対応する必要があります。

Stripeを利用している場合、これらの対応についても「URL１つ」で解決できます。

## カスタマーポータルで、顧客の契約管理機能をノーコードに提供しよう

ダッシュボード上部の[歯車]アイコンをクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/33b810652885-20230526.png)
[Billing]セクションの[カスタマーポータル]をクリックします。
![](https://storage.googleapis.com/zenn-user-upload/c184adbbfb68-20230526.png)

顧客が確認・操作したい機能やデータをまとめて見ることができる、[カスタマーポータル]の設定画面が開きます。

![](https://storage.googleapis.com/zenn-user-upload/62805f50a4f7-20230526.png)

このURLを、アプリケーションに追加しましょう。


`src/pages/index.astro`に`a`タグでURLを追加します。

```diff html:src/pages/index.astro
---
import Layout from '../layouts/Layout.astro';
import Card from '../components/Card.astro';
---

<Layout title="Welcome to Astro.">
+	<header>
+		<a href="https://billing.stripe.com/p/login/test_xxxxk">マイページ</a>
+	</header>
	<main>
	<main>
		<script async
		  src="https://js.stripe.com/v3/buy-button.js">
		</script>
		
		<stripe-buy-button
		  buy-button-id="buy_btn_xxxxx"
		  publishable-key="pk_test_xxxxxx"
		>
		</stripe-buy-button>
	</main>
</Layout>
```

ファイルを保存すると、マイページへのリンクテキストが表示されます。

![](https://storage.googleapis.com/zenn-user-upload/68e6ecad8d60-20230623.png)

このリンクにアクセスすると、メールアドレスを利用した認証ページが開きます。

![](https://storage.googleapis.com/zenn-user-upload/c6ef9b1105de-20230526.png)

## テスト注文と、マイページへのアクセスを試してみよう

これでサブスクリプションの申し込みと、申し込み情報の管理の準備ができました。

ここで一度どのような申し込みフローになるかを試してみましょう。

### [購入ボタン]をクリックして、申し込みページに移動する

まずは申し込みから始めましょう。

Astroアプリケーションに埋め込んだ[購入ボタン]をクリックして、申し込みページに移動します。

![](https://storage.googleapis.com/zenn-user-upload/e8e29a54f9aa-20230623.png)

申し込みページでは、メールアドレスやテストのカード番号を入力して申し込みを完了しましょう。

![](https://storage.googleapis.com/zenn-user-upload/12d65cff0056-20230623.png)

### テストのカード情報で、申し込みを完了しよう

次の表に従って項目を入力してください。

|項目|入力値|備考|
|:--|:--|:--|
|メールアドレス|<Stripeダッシュボードへのログインに利用しているメールアドレス>|テストモードでは、このアドレス以外にはメールが送信されません。|
|カード番号|4242424242424242||
|有効期限|12(月) / 40(年)|未来の月と年を入力しましょう。|
|セキュリティコード|111|テストモードでは、３桁の数字ならなんでもOKです。|
|カード所有者名|<あなたのお名前>||
|「情報を〜」|オフのまま||

入力が完了したら、[申し込む]ボタンをクリックします。

![](https://storage.googleapis.com/zenn-user-upload/ba0670a52c1d-20230623.png)

完了画面が表示されれば、サブスクリプションの契約申し込みは完了です。

![](https://storage.googleapis.com/zenn-user-upload/76989d20a587-20230623.png)


:::message
カード決済が失敗したケースや不審請求などのシミュレーションも、特定のカード番号でテストできます。

https://stripe.com/docs/testing#cards

:::

### 顧客マイページにアクセスしてみよう

申し込みが完了したので、マイページに移動しましょう。

Astroアプリケーションに追加した、[マイページ]リンクをクリックします。

![](https://storage.googleapis.com/zenn-user-upload/68e6ecad8d60-20230623.png)

メールアドレスを利用した認証ページが開きました。

ここで先ほどの申し込みで入力したメールアドレスを入力しましょう。

![](https://storage.googleapis.com/zenn-user-upload/c6ef9b1105de-20230526.png)

認証メールを送信したことが、画面に表示されます。

![](https://storage.googleapis.com/zenn-user-upload/db537e10b467-20230623.png)

お使いのメールアプリ・クライアントで、ログイン用のメールが届いているか確認しましょう。

![](https://storage.googleapis.com/zenn-user-upload/5ee7e753d029-20230623.png)

ログインボタンをクリックすると、マイページが表示されます。

![](https://storage.googleapis.com/zenn-user-upload/dfc59bd91a3d-20230623.png)

### [Tips] 顧客に触らせたい機能は、変更可能

顧客のマイページでは、「履歴確認や領収書のDLだけ許可したい」から「カスタマーポータルでできることは、全部やらせたい」まで要件がさまざまです。

![](https://storage.googleapis.com/zenn-user-upload/2ba6f5cc7e94-20230526.png)

カスタマーポータルの編集画面では、機能ごとにオン・オフが変更できますので、仕様や要件に合わせてカスタマイズしましょう。

![](https://storage.googleapis.com/zenn-user-upload/fa421792cb5a-20230526.png)


## Checkpoint

- サブスクリプションの情報管理は、カスタマーポータルで
- テスト用のカード番号を使って、様々な支払いシーンをテストしよう
- テスト環境では、ログイン中のメールアドレスを使ってメール送信をテストしよう