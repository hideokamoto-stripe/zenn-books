---
title: "Next.jsのダイナミックルーティングで、商品詳細ページを実装しよう"
---

ここまで、商品一覧ページ１枚での組み込みを行いました。
このステップでは、ECサイトのように個別の商品詳細ページについても組み込んでいきましょう。

## `pages/products/[id].jsx`で商品詳細ページを用意しよう

## `pages/api/products/[id].js`で商品詳細用のAPIを実装しよう

## 商品詳細ページで、商品情報を表示させよう

## `useShoppingCart`でカート機能を実装しよう

## おさらい

- `pages/[id].jsx`で動的にページを生成できる
- `pages/api/[id].js`でAPIのパスパラメータを設定できる
- use-shopping-cartは、ページを跨いだ場合でもカートの中身を共有できる

最後のステップでは、顧客が注文した内容をWebhookで受け取る方法を紹介します。

##　「早く終わった」という方のための、もう１ステップ

`getStaticPaths`メソッドを利用すると、動的なページを事前にビルドすることができます。

公式ドキュメントを参考に、商品詳細ページを事前にビルドさせてみましょう。

https://nextjs.org/docs/basic-features/data-fetching/get-static-paths