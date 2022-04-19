---
title: "use-shopping-cartとStripe SDKで簡単な決済フォームを追加しよう"
---

ここからは、表示した商品の注文ボタンの実装を行います。

## Stripe Checkoutのクライアント側組み込みを有効化しよう

## use-shopping-cartをインストール・セットアップしよう

## `useShoppingCart`フックで、「いますぐ注文」ボタンを実装しよう

### `checkoutSingleItem`
## APIを利用し、より高機能な決済フォームに切り替えよう

## おさらい

- Stripe Checkoutは、クライアント側だけで決済フォームの設定ができる
- use-shopping-cartで、Stripeまわりの操作ができるReact Hookが使える
- 送料やクーポン入力の設定など、より多くの機能を使うにはサーバー側の処理が必要

ここまでで、商品毎に注文を行う機能の実装に成功しました。
次のステップでは、複数の商品をまとめて注文できる「カート機能」を実装します。

##　「早く終わった」という方のための、もう１ステップ

Stripe Checkoutのセッション作成APIには、さまざまなオプションが用意されています。

ドキュメントを参考に、`expires_at`などの未紹介パラメーターについても試してみましょう。

https://stripe.com/docs/api/checkout/sessions/create

また、以下の記事を参考に、「その場限りの料金」での決済ページを作る方法についてもチャレンジしてみましょう。

https://qiita.com/hideokamoto/items/6bd31422e2ebb27d4eaa