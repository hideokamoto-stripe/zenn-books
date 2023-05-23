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

#### Step　1: 連結アカウントを作成する

まずは連結アカウントを作成します。

作成する際に、アカウントの種類を指定しましょう。

```javascript
const account = await stripe.accounts.create({type: 'standard'});
```

出店者のビジネスサイトやサポート電話番号、郵送先住所などが前もってわかっている場合には、APIの引数を利用して事前入力ができます。

事前に設定しておくことで、出店者がこれらの情報を再び入力しなくても済むようにしましょう。

https://stripe.com/docs/api/accounts/create

#### Step　2: アカウントへのリンクを作成する

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

### コンプライアンス要件の変化に自動で対応する

グローバルなルールやコンプライアンス要件は、時間の経過とともに進化し続けます。

StandardアカウントまたはExpressアカウントでは、Stripeが追加の提出が必要になったアカウント情報を積極的に収集します。

Customアカウントのうち、独自のオンボーディングフローを構築していて、[Connect Onboarding](https://stripe.com/jp/connect/onboarding)を利用していない場合には、ルールやコンプライアンス要件が変化する度にフローの変更作業が必要となります。

ただし、[Connect Onboarding](https://stripe.com/jp/connect/onboarding)を利用することで、オンボーディングや認証情報の収集と、フォームの更新作業を自動化することができます。
また、[Connect Onboarding](https://stripe.com/jp/connect/onboarding)を使っている場合、コンプライアンス要件の変化にも追加の作業を必要とせずに対応できるようになります。

コンプライアンス要件の変化に伴う変更をユーザーに伝える方法については、Customアカウント向けのベストプラクティスを参考にしてください。

https://support.stripe.com/questions/best-practices-for-connect-platforms-communicating-updates-to-verification-requirements-with-custom-connected-accounts?locale=ja-JP

「組み込みに必要な工数」は、Stripe Connectのアカウントタイプを選択する際の重要な最も要素の1つです。
ほとんどのスタートアップや成長企業においては、実装と運用保守のためのリソースはあまり多くありません。
Customアカウントの実装および運用保守に、多くの開発リソースを用意できる場合を除いて、StandardまたはExpressを選ぶことをお勧めします。

## 運用上の責任
もう1つの重要な考慮事項は、「日々の運用責任を誰が受け持つか」です。
例えば、このような運用事項を考慮する必要があります。

- プラットフォームは連結アカウントで発生した支払いに対する不正利用や不審請求の申立てに責任を負いますか？
- 連結アカウントのユーザーは、どのダッシュボードを利用して、支払いの表示や払い戻し処理、不審請求の申立ての対応そしてアカウント情報の更新などを行いますか？
- 連結アカウントのユーザーは、誰に対して問題の解決などのサポートを求めることができますか？

一部のケースでは、連結アカウントへの支払い方法（ダイレクト支払い・デスティネーション支払い）によって異なります。
支払い方法の違いについては、別の記事にて紹介しますが、支払い方法によって動作要件が変わることがある点にも注意が必要です。

### 不正利用や不審請求の申立てに対する責任

StandardアカウントかExpressアカウントかで悩んでいる場合には、最終的な決め手になるかもしれません。

Standardアカウントでダイレクト支払いを利用している場合は、連結アカウントのユーザーが不正利用や不審請求の申立てに対応する責任を負います。
ExpressアカウントやCustomアカウントでは、推奨された支払いタイプ（デスティネーション支払い）を利用する場合はプラットフォームを提供する側が最終的な責任を負います。

不審請求申立てへの対応プロセスや不正利用への対処方法を理解することも、オンラインビジネスの重要な要素です。
Standardアカウントでは連結アカウントユーザーにも対応のための知識が必要ですが、Expressアカウントではプラットフォーム側に任せることができます。

https://stripe.com/docs/connect/charges#%E4%B8%8D%E5%AF%A9%E8%AB%8B%E6%B1%82%E3%81%AE%E7%94%B3%E8%AB%8B%E3%81%A8%E3%83%81%E3%83%A3%E3%83%BC%E3%82%B8%E3%83%90%E3%83%83%E3%82%AF

### ダッシュボードへのアクセス

Standardアカウントでは、連結アカウントのユーザーがStripeダッシュボードにログインして請求を処理できます。
何らかの理由で、連結アカウントのユーザーとエンドユーザーの関係を難読化したい場合には、Standardアカウント以外を選ぶ必要があります。

![](https://res.cloudinary.com/practicaldev/image/fetch/s--OHZCF_a4--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/w1v1s4erk7nntp3q1gsu.png)

Expressアカウントでは、Stripeが提供する「Expressダッシュボード」を連結アカウントのユーザーに提供します。
Expressダッシュボードから、連結アカウントのユーザーはアカウント残高や今後の入金予定、支払い・収益の追跡などができます。
ただし、Expressダッシュボードには、通常のStripeダッシュボードと利用できる機能が同じではありません。

![](https://res.cloudinary.com/practicaldev/image/fetch/s--JYd70shY--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3dkajjto20x74nuh6eab.png)

Customアカウントを利用すると、連結アカウントのユーザーはStripeがホストするダッシュボードを利用できません。
ユーザーがビジネスの運営に関連するタスクを処理できるように、プラットフォーム側で機能やUIを提供しましょう。

### ユーザーサポート

Customアカウントでは、連結アカウントに関する質問はすべてプラットフォーム側で対応する必要があります。
Standard・Expressアカウントでは、一部の質問についてはStripe側がプラットフォームに代わって回答します。
例えば、本人確認やコンプライアンスの要件が変化した場合、Stripeから連結アカウントのユーザーに連絡をとり、アカウント情報が現地の規制要件に準拠した内容になっていることを確認します。

## どのアカウントタイプが最適か？

独自のインテグレーションが必要な強力な理由や、独自のUIやフローを構築・維持するための多くのリソースがない場合、StandardアカウントまたはExpressアカウントを選ぶでしょう。

構築しているプラットフォームが不正利用や不審請求の申立てを対処できると仮定すると、StandardアカウントとExpressアカウントどちらを選ぶかは、連結アカウントのユーザーがエンドユーザーとどのような関係を持つかによって決まります。
プラットフォームのユーザー、つまり、連結アカウントユーザーは、顧客と長期的な関係を築く必要があるのか、それとも一度きりの取引的な関係になるでしょうか？
もしそうである場合、顧客が不正利用や不審請求の申立てを自分で対処できるだけの知識がある（または学べる）と期待するか、エンドユーザーに対して手動で課金する能力を信頼しているかどうかを考慮しましょう。

オンラインビジネスに精通した人々と働いていて、エンドユーザーと長期的な関係を維持している場合、Standardアカウントを選びます。
一度だけの取引のあるエンドユーザーと関わる一般のインターネットユーザーや、エンドユーザーへの課金を防ぎたい場合には、Expressアカウントに傾くでしょう。

## 元記事: Standard vs. Express vs. Custom account types for Stripe Connect

https://dev.to/stripe/standard-vs-express-vs-custom-account-types-for-stripe-connect-386j