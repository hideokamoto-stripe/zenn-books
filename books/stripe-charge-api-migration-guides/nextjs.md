---
title: "Next.jsでの、Payment Intent切り替えガイド"
---

この章では、Next.jsを利用した構成での、マイグレーション方法を紹介します。

## Step0: Charge / Token APIを利用したサンプルアプリ

コードの差分を示しながら紹介するため、まずは古いAPIを利用して実装します。

開発環境については、フロントエンド・バックエンドをまとめて管理するため、便宜上Nxを利用します。

```bash
$ yarn create next-app --typescript
yarn create v1.22.17
[1/4] 🔍  Resolving packages...
[2/4] 🚚  Fetching packages...
[3/4] 🔗  Linking dependencies...
[4/4] 🔨  Building fresh packages...

success Installed "create-next-app@12.3.0" with binaries:
      - create-next-app
✔ What is your project named? … example-nextjs
```

## Step1: Stripe SDKのアップグレード

