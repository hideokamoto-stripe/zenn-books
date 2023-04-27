---
title: "Standard vs. Express vs. Custom account types for Stripe Connect"
---

Stripe Connectを使用する場合、プラットフォームの中で支払いを受け取ったり、支払いを行ったりするためのユーザーアカウント（「連結アカウント」とよびます）を作成する必要があります。

ユーザーがプラットフォームに参加する（サインアップ）する都度、この連結アカウントを作成します。

プラットフォームはユーザーに提供するアカウントタイプを、Standard・Express・Customの３つから選ぶことができます。

どの種類を選ぶかによって、プラットフォームにStripeを組み込む作業の量や、対応すべき運用上の責任（チャージバックの処理やユーザーのサポートなど）が決まります。

この3つのアカウントタイプは、異なるユースケースやセグメントに対応するため、どのタイプが自身のプロジェクトにマッチしているかを判断する必要があります。

この記事では、統合の手間や運用上のオーバーヘッドを比較して、いくつかの推奨事項を紹介します。

:::message
💰 ExpressまたはCustomアカウントタイプを使用する場合には、追加費用がかかります。
料金の詳細は、以下のページでご確認ください。

https://stripe.com/jp/connect/pricing

:::

## 元記事: Standard vs. Express vs. Custom account types for Stripe Connect

https://dev.to/stripe/standard-vs-express-vs-custom-account-types-for-stripe-connect-386j