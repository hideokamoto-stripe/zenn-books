---
title: "Standard vs. Express vs. Custom account types for Stripe Connect"
---

Stripe Connectを使用する場合、プラットフォームの中で支払いを受け取ったり、支払いを行ったりするための出店者アカウント（「連結アカウント」とよびます）を作成する必要があります。

出店者がプラットフォームに参加する（サインアップ）する都度、この連結アカウントを作成します。

プラットフォームは出店者に提供するアカウントタイプを、Standard・Express・Customの３つから選ぶことができます。

どの種類を選ぶかによって、プラットフォームにStripeを組み込む作業の量や、対応すべき運用上の責任（チャージバックの処理やエンドユーザー・出店者のサポートなど）が決まります。

この3つのアカウントタイプは、異なるユースケースやセグメントに対応するため、どのタイプが自身のプロジェクトにマッチしているかを判断する必要があります。

この記事では、統合の手間や運用上のオーバーヘッドを比較して、いくつかの推奨事項を紹介します。

:::message
💰 ExpressまたはCustomアカウントタイプを使用する場合には、追加費用がかかります。
料金の詳細は、以下のページでご確認ください。

https://stripe.com/jp/connect/pricing

:::

## 組み込みの難易度を比較する

まずは組み込みの難易度で比較してみましょう。

簡単に組み込みができるのは、StandardとExpressの２つです。

Standardアカウントを利用すると、組み込みに必要な作業がもっとも少なくなります。

ただしユーザー体験をより細かく・柔軟に制御したい場合には、Customアカウントを利用する必要があります。

Customアカウントの実装作業は少なくありませんが、自社のブランドに沿った（ホワイトラベル化）した体験を提供することができます。

各アカウント タイプの統合要件の詳細は次のとおりです。

### オンボーディング

出店者がプラットフォームに参加する際の、オンボーディング画面を見ていきましょう。

全てのアカウントタイプで、Stripeがホストする「[Connect Onboarding](https://stripe.com/jp/connect/onboarding)」を利用できます。

https://stripe.com/jp/connect/onboarding

[Connect Onboarding](https://stripe.com/jp/connect/onboarding)を使用するには、少なくともAPIを2つ呼び出す必要があります。

#### Step1: 連結アカウントを作成する

まずは連結アカウントを作成します。

作成する際に、アカウントの種類を指定しましょう。

```javascript
const account = await stripe.accounts.create({type: 'standard'});
```

出店者のビジネスサイトやサポート電話番号、郵送先住所などが前もってわかっている場合には、APIの引数を利用して事前入力ができます。

事前に設定しておくことで、出店者がこれらの情報を再び入力しなくても済むようにしましょう。

https://stripe.com/docs/api/accounts/create

#### Step2: アカウントへのリンクを作成する

作成した連結アカウントのIDを利用して、ホスト型のオンボーディング画面に移動するためのURLを取得します。

```javascript
const accountLink = await stripe.accountLinks.create({
  account: account.id,
  refresh_url: 'https://example.com/reauth',
  return_url: 'https://example.com/return',
  type: 'account_onboarding',
});
// redirect to accountLink.url
```

https://stripe.com/docs/api/account_links/create

Stripeが用意するオンボーディングUIを利用することで、出店者がプラットフォームに簡単にオンボードできます。
本人確認機能やオンボーディングフローを自社で開発する必要がなくなり、出店者もアカウントのセットアップをスムーズに進められます。

全てのアカウントタイプで、この[Connect Onboarding](https://stripe.com/jp/connect/onboarding)利用することをお勧めします。

もちろんCustomアカウント向けに、独自のオンボーディングフローを実装することも可能です。

追加の確認を行うステップを追加したり、アカウントに関する情報の更新や独自のフォーム利用することなどもできます。

ただし本人確認のステップや国ごとのローカライズなどへの対応コストを考えると、少なくない開発リソースが必要です。

したがって、エンジニアのリソースに余裕がある場合以外では、[Connect Onboarding](https://stripe.com/jp/connect/onboarding)を利用しましょう。


### Automatic updates for new compliance requirements
Global compliance requirements and rules do change over time. For Standard and Express accounts, Stripe will proactively reach out and collect information about your accounts whenever requirements change.

If you’re using Custom account types with a bespoke onboarding flow and haven’t integrated with Connect Onboarding, you’ll need to invest development resources to make updates as compliance requirements change over time. However, if you use Connect Onboarding for Custom accounts, you can easily collect onboarding and verification information from your users and save yourself the hassle of constantly updating your onboarding form. Plus, Connect Onboarding can help you stay up-to-date with the latest compliance requirements without any extra effort on your part. Just make sure to follow best practices for communicating changes to your users, as outlined in the guide for Custom accounts.

Integration effort is one the most heavily weighted factors when deciding which Stripe Connect account type to choose. Most startup and growth companies have the bandwidth and resources to implement and maintain Standard and Express accounts. If you don’t have a team of engineers to dedicate significant bandwidth to maintaining a Custom integration, I recommend Standard or Express.

## Operational responsibilities
Another important consideration is who handles the operational responsibility day to day. For instance, are you the platform liable for fraud and disputes related to payments for a connected account? What type of dashboard can the connected account access to view payments, issue refunds, handle disputes, and update their information? Who is responsible for supporting connected account users when questions arise? Some of the following details differ depending on the charge type (direct vs destination) which we’ll cover in another article. Just know that one charge type may have different operational requirements than another charge type.

### Fraud and dispute liability
If you’re trying to decide between Standard and Express accounts, this is likely the tie breaker.

For Standard accounts, the connected account owner (your user) is responsible for dealing with fraud and disputes*. For Express and Custom accounts, you as the platform are ultimately responsible for dealing with fraud and disputes.

Understanding the dispute process and how to handle fraud is part of any online business. Standard accounts are more suited for experienced online businesses, whereas Express accounts are ideal for businesses at any level of experience.

*this may shift depending on the charge type used.

### Dashboard access
Standard accounts are conventional Stripe accounts where the account holder (your user) is able to log in to the Dashboard and can process charges on their own. If you want to obfuscate the relationship between the connected account holder and their end user for any reason, Standard is likely not a good fit.

![](https://res.cloudinary.com/practicaldev/image/fetch/s--OHZCF_a4--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/w1v1s4erk7nntp3q1gsu.png)

Express accounts can only use the Express Dashboard, a simplified Stripe-hosted interface. From the Express Dashboard, your users can view their available balance, see upcoming payouts, and track their earnings in real time. The Express Dashboard does not have all of the same features as the Standard Dashboard.

![](https://res.cloudinary.com/practicaldev/image/fetch/s--JYd70shY--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3dkajjto20x74nuh6eab.png)

Custom accounts do not have access to a Stripe-hosted dashboard. You’ll need to build custom interfaces into your application for your users to handle any operational tasks related to running their business.

### User support
All support questions about a Custom connected account are handled by you, the platform. For Standard and Express accounts, some questions can be answered by Stripe and others will be handled by the platform. For instance, as identity verification requirements shift and change, Stripe will proactively reach out to your users to ensure the information on file stays compliant with local regulations.

## Recommendations
Absent a compelling reason for a bespoke integration and lots of resources to build and maintain a Custom Connect integration, I would default to either Standard or Express. Assuming the platform I’m building is open to handling fraud and disputes, the platform decision between Standard and Express comes down to the relationship I expect users to have with end customers. Will my users, the owners of the connected accounts, need a long standing relationship with their customers or will their relationship be one-off, and transactional in nature? If so, do I expect my customer base to be savvy enough (or teachable enough) to handle fraud and disputes and trusted with the ability to charge the end customer manually. If I’m working with online business savvy folks who maintain long-running relationships with their end customers, I’d choose Standard. If I’m working with any internet user who has a one-off relationship with end customers or if I want to prevent them from charging end customers, then I’d lean towards Express.

![](https://res.cloudinary.com/practicaldev/image/fetch/s--lDe7s6rK--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/u8t0gbpesgerk30xxce6.png)

## 元記事: Standard vs. Express vs. Custom account types for Stripe Connect

https://dev.to/stripe/standard-vs-express-vs-custom-account-types-for-stripe-connect-386j