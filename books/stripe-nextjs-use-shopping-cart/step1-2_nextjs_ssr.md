---
title: "Next.jsでAPIやサーバー側処理のあるサイトを作ってみよう"
---

続いてAPIも利用する場合の開発方法を簡単に体験しましょう。

## Next.jsのREST APIを動かしてみよう

Next.jsでAPIを開発する場合、`pages/api`ディレクトリにファイルを配置します。

`create-next-app`を利用した場合、`pages/api/hello.js`がサンプルとして用意されています。

**pages/api/hello.js**

```js
// Next.js API route support: https://nextjs.org/docs/api-routes/introduction

export default function handler(req, res) {
  res.status(200).json({ name: 'John Doe' })
}
```

`curl http://localhost:3000/api/hello`をターミナルで実行して、APIを動かしてみましょう。

コードに記述された、`{"name":"John Doe"}`が表示されます。

```bash
curl http://localhost:3000/api/hello
{"name":"John Doe"}
```

ここで、`pages/api/hello.js`を以下のように変更してみましょう。

```diff js
// Next.js API route support: https://nextjs.org/docs/api-routes/introduction

export default function handler(req, res) {
-  res.status(200).json({ name: 'John Doe' })
+  res.status(200).json({ name: 'John Doe', age: 20})
}
```

もう一度cURLを実行すると、APIのレスポンスが変わっていることが確認できます。

```bash
curl http://localhost:3000/api/hello
{"name":"John Doe","age":20}
```

## 簡単な商品データを返すAPIを実装しよう

新しいAPIを追加する場合、そのAPIのパスに応じたファイルを追加します。

`pages/api/products.js`を新しく作成しましょう。

ファイル内のコードは、以下のサンプルを利用します。

**pages/api/products.js**

```js
export default function handler(req, res) {
    // GET以外のリクエストを許可しない
    if (req.method.toLocaleLowerCase() !== 'get') {
        return res.status(405).end()
    }
    res.status(200).json([{
        name: "胡麻鯖セット",
        price: 5000,
    }, {
        name: "明太子詰め合わせ",
        price: 6000
    }])
}
```

`res.json()`の中にサイトで表示させたい商品データを配列で記述しています。
こちらもcURLを実行すると、先ほど記述したデータが返ってきます。

```bash
curl   http://localhost:3000/api/products
[{"name":"胡麻鯖セット","price":5000},{"name":"明太子詰め合わせ","price":6000}]
```

### `req.method.toLocaleLowerCase() !== 'get'`がやっていること
`req.method.toLocaleLowerCase()`にて、APIが呼び出したメソッド(`GET` / `POST` / etc..)を取得しています。
今回は「商品データの取得」が目的ですので、`GET`以外のリクエストではエラーを返します。

`res.status()`で405を指定することで、`HTTP 405 Method Not Allowed`を伝えることができます。
また`end()`でレスポンスbodyを持たないことを定義しました。
これによって、cURLでGET以外のリクエストを送ると、エラーが返ってきます。

```bash
curl -I  http://localhost:3000/api/products -XPOST 
HTTP/1.1 405 Method Not Allowed
Date: Tue, 19 Apr 2022 04:25:45 GMT
Connection: keep-alive
Keep-Alive: timeout=5
Transfer-Encoding: chunked
```

## Next.jsの`getServerSideProps`でAPIのデータを取得しよう

ここからは、APIのデータを利用して、コンテンツを作成する方法を体験します。

Next.jsでは、`geetServerSideProps`または`getStaticProps`関数をexportすることで、リクエストごとまたはビルドごとにサーバー側で処理を行うことができます。

```js
export async function getServerSideProps (context) {
    return {
        props: {
            resolvedUrl: context.resolvedUrl,
        }
    }
}
```

`geetServerSideProps`または`getStaticProps`関数は、`pages/`ディレクトリにあるファイルでexportする必要があります。
コンポーネント単位で処理を定義することはできない点に注意しましょう。

### fetchを利用して、APIのデータを取得する

`getServerSideProps`の中では、`http`から始まる絶対URLを利用してAPIを呼び出すことができます。

`pages/hello-world.jsx`に、先ほど作成したAPIを呼び出すコードを追加しましょう。

```jsx
export async function getServerSideProps (context) {
    try {
        const host = context.req.headers.host || 'localhost:3000'
        const protocol = /^localhost/.test(host) ? 'http' : 'https' 
        const products = await fetch(`${protocol}://${host}/api/products`)
            .then(data => data.json())
        return {
            props: {
                products,
            }
        }
    } catch (e) {
        console.log(e)
        return {
            props: {
                products: [],
            }
        }
    }
}
```

http://localhost:3000/hello-world　にアクセスして、ターミナルにエラーが表示されなければOKです。

### `getServerSideProps`で取得したデータをReactで表示しよう

続いて、取得したデータをReactで表示させます。

`pages/hello-world.jsx`のコードを以下のように変更しましょう。


```diff jsx
-export default function HelloWorld() {
+export default function HelloWorld(props) {
    return (
        <div>
            <Head>
                <title>Hello World</title>
                <meta name="description" content="検索エンジン用の説明文" />
            </Head>
            <h1>Hello World</h1>
+            <pre><code>{JSON.stringify(props,null,2)}</code></pre>
        </div>
    )
}

```

http://localhost:3000/hello-world　にアクセスして、APIのレスポンスデータが表示されなければOKです。

![](https://storage.googleapis.com/zenn-user-upload/597376dc818f-20220419.png)


## `npx next build`でアプリをビルドしよう

サーバー側処理を含む場合でも、静的化する場合と同様にアプリのビルドが必要です。

ターミナルで`npx next build`を実行しましょう。

```bash
npx next build

info  - Checking validity of types  
info  - Creating an optimized production build  
info  - Compiled successfully
info  - Collecting page data  
info  - Generating static pages (4/4)
info  - Finalizing page optimization  

Page                                       Size     First Load JS
┌ ○ /                                      962 B          84.5 kB
├   └ css/149b18973e5508c7.css             655 B
├   /_app                                  0 B            83.6 kB
├ ○ /404                                   192 B          83.8 kB
├ λ /api/hello                             0 B            83.6 kB
├ λ /api/products                          0 B            83.6 kB
└ λ /hello-world                           445 B            84 kB
+ First Load JS shared by all              83.6 kB
  ├ chunks/framework-fc97f3f1282ce3ed.js   44.9 kB
  ├ chunks/main-f4ae3437c92c1efc.js        28.3 kB
  ├ chunks/pages/_app-4897a2087ece2691.js  9.59 kB
  ├ chunks/webpack-69bfa6990bb9e155.js     769 B
  └ css/0343dc3e6093f50e.css               23.7 kB

λ  (Server)  server-side renders at runtime (uses getInitialProps or getServerSideProps)
○  (Static)  automatically rendered as static HTML (uses no initial props)
```

### Tips: `getServerSideProps`があると、`next export`はできない

`getServerSideProps`関数は、リクエストの度にサーバー側で処理を行います。

しかし`next export`した静的サイトでは、サーバー側の処理を実行することができません。

そのため、`getServerSideProps`を含むpageが存在している場合、`next export`は失敗します。

```bash
npx next export

Error occurred prerendering page "/hello-world". Read more: https://nextjs.org/docs/messages/prerender-error
Error: Error for page /hello-world: pages with `getServerSideProps` can not be exported. See more info here: https://nextjs.org/docs/messages/gssp-export
```

ビルド結果の`Page`に`λ`がついているページは、サーバー側の処理を必要としています。
そのため、`next export`を利用したい場合は、`λ`のついているページがないかを確認しましょう。

## `npx next start`でビルドしたアプリを実行しよう

ビルドしたアプリを実行するには、`npx next start`コマンドを実行します。

```bash
npx next start

ready - started server on 0.0.0.0:3000, url: http://localhost:3000
```

`ready`のメッセージが表示されると、 http://localhost:3000 からビルドしたアプリにアクセスできます。

### Tips: `next start`はポート3000を利用する

`next dev`を実行している場合や、別のアプリをローカルで起動している場合、以下のエラーが出ることがあります。

```bash
npx next start

Error: listen EADDRINUSE: address already in use 0.0.0.0:3000
    at Server.setupListenHandle [as _listen2] (net.js:1331:16)
    at listenInCluster (net.js:1379:12)
    at doListen (net.js:1516:7)
    at processTicksAndRejections (internal/process/task_queues.js:83:21) {
  code: 'EADDRINUSE',
  errno: -48,
  syscall: 'listen',
  address: '0.0.0.0',
  port: 3000
}
```

これは、「すでにポート3000が使われているので、サーバーを起動できない」ことを知らせています。

もしこのエラーが表示された場合は、`npx next start -p 3100`のように、別のポート番号を指定しましょう。

```bash
npx next start -p 3100
ready - started server on 0.0.0.0:3100, url: http://localhost:3100
```


## 「早く終わった」という方のための、もう１ステップ

APIエラーが発生した場合、デモコードではエラーを握りつぶして、代わりに「商品数０件」として結果を返します。

しかし実際の案件では、適切なエラーメッセージや、フォールバックを行う必要があります。

Next.jsのエラーページカスタマイズの記事などを参考に、APIがエラーを返したときの処理の実装に挑戦してみましょう。

https://nextjs.org/docs/advanced-features/custom-error-page#500-page