---
title: "サブスクリプション申し込みフォームをカスタマイズしよう"
---

Stripeのノーコード・ローコード機能の特徴の１つは、「埋め込みコードを触らずに、見た目や設定をカスタマイズできる」ことです。

ここでは３つのカスタマイズを紹介します。

## 商品を変更しよう

提供したい商品やサービスを変更したい場合、URLや埋め込みコードを変更せずに、商品の入れ替えだけで対応できます。

Stripeダッシュボードで、[支払い]から[支払いリンク]に移動しましょう。

![](https://storage.googleapis.com/zenn-user-upload/8485235fd4a5-20230623.png)
[コーヒー豆定期お届け便]のリンクを選びます。
![](https://storage.googleapis.com/zenn-user-upload/2da6c3093093-20230623.png)

[購入ボタン]ボタン横の[...]をクリックし、[編集]を押しましょう。

![](https://storage.googleapis.com/zenn-user-upload/4a842bfcba39-20230623.png)

[商品]セクションに[別の商品を追加]テキストがあります。これをクリックしましょう。

![](https://storage.googleapis.com/zenn-user-upload/a1b1c14cbdf7-20230623.png)

登録済みの商品を選択するか、新しく商品を追加することができます。

今回は例として、登録済みの別の商品を選択しました。

![](https://storage.googleapis.com/zenn-user-upload/baeed41c6a6c-20230623.png)

追加すると、商品が２つになります。

![](https://storage.googleapis.com/zenn-user-upload/e5618393b8e0-20230623.png)

この状態で、もともとあった商品エリアにある[...]ボタンをクリックしましょう。

[商品を削除]ボタンが表示されますので、クリックします。

![](https://storage.googleapis.com/zenn-user-upload/5537976713a4-20230623.png)

商品が入れ替わりました。この状態で変更を保存しましょう。

![](https://storage.googleapis.com/zenn-user-upload/ed85a8ad586d-20230623.png)

Astroに埋め込んだ購入ボタンの表示などが変わりました。

![](https://storage.googleapis.com/zenn-user-upload/08f8e02e99e2-20230623.png)

このように、商品やサービスの入れ替えについても、コードを書かずに対応できます。

## 「トライアル期間」を設定しよう

食品などの物理的なアイテムの定期販売では難しいですが、個人レッスンやオンラインサービスでは「まずは7日間お試しを」などのオファーを出すことができます。

支払いリンクを作成・編集する際に、[オプション]で[無料トライアルを含める]をオンにしましょう。
![](https://storage.googleapis.com/zenn-user-upload/9c75aa0aaa79-20230526.png)

日数を入力できますので、[7]や[14]などを入れましょう。
![](https://storage.googleapis.com/zenn-user-upload/ea2282bf1bd3-20230526.png)

リンクを更新すると、決済画面に[無料期間]の表示が追加されます。

![](https://storage.googleapis.com/zenn-user-upload/c494df7d6e8e-20230526.png)

## 　「初回契約手数料」などの、「１回きりの請求」を一緒に行う方法

ビジネスモデルや事務手続の関係で、初回の契約時のみ追加の請求を行いたいことがあります。

その場合は、支払いリンクの商品に「サブスクリプションの料金」と「１回きりの商品」両方を登録しましょう。

![](https://storage.googleapis.com/zenn-user-upload/6def78011d07-20230526.png)

決済ページでも、「初回のみXX円、次回からはYY円」と表示されます。

![](https://storage.googleapis.com/zenn-user-upload/27ac0a80572b-20230526.png)

### 購入ボタンの見た目を変更してみよう

ボタンのみ埋め込みしたい場合は、[設定]を[カード]から[ボタン]に変更します。

![](https://storage.googleapis.com/zenn-user-upload/a1c20cd8cc88-20230623.png)


また、配色やボタンテキストのカスタマイズも行えます。

![](https://storage.googleapis.com/zenn-user-upload/6c1a84959f2d-20230623.png)

変更の保存と反映は、パネル下部にある[変更を保存してコードをコピーする]ボタンをクリックします。

コードスニペットが変わることはないため、埋め込み済みの購入ボタンにも変更を反映できます。

## Checkpoint

- １度埋め込んだリンクやボタンは、再利用できる
- トライアルや初回手数料の請求も、ノーコードで
- 見た目の変更も、ダッシュボード操作だけである程度可能