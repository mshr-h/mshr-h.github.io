# Next.js Learn (Excel)で静的コンテンツ生成などを学んだ


以前、[Next.js Learn (Basics)でNext.jsの基礎を学んだ](https://keepcodingkeepclimbing.com/blog/nextjs-learn-basics)でNext.jsのチュートリアルを実施した。今回は続きの[Next.js Learn (Excel)](https://nextjs.org/learn/excel/static-html-export)を実施したので、まとめた。

# 内容

Next.js Learn (Excel)は4つのレッスンで構成されている。

- [Export into a Static HTML App](https://nextjs.org/learn/excel/static-html-export)
- [TypeScript](https://nextjs.org/learn/excel/typescript)
- [Create AMP Pages](https://nextjs.org/learn/excel/amp)
- [Automatic Static Optimization](https://nextjs.org/learn/excel/automatic-static-optimization)

# Export into a Static HTML App

Next.js Learn (Basics)で作成したTV Showsを静的コンテンツとしてエクスポートする。TV Showsのルートディレクトリに下記内容を`next.config.js`として保存する。これは、エクスポート時に呼ばれる関数で、`/`と`/about`、`/show/[id]`のパス情報を出力する。`/show/[id]`はTVmaze APIのアクセス結果からパス情報を取得している。

```jsx
const fetch = require('isomorphic-unfetch');

module.exports = {
    exportTrailingSlash: true,
    exportPathMap: async function() {
        const paths = {
            '/': { page: '/' },
            '/about': {page: '/about'}
        };
        const res = await fetch('https://api.tvmaze.com/search/shows?q=batman');
        const data = await res.json();
        const shows = data.map(entry => entry.show);

        shows.forEach(show => {
            paths[`/show/${show.id}`] = {page: '/show/[id]', query: {id: show.id } };
        });

        return paths;
    }
};
```

ルートディレクトリで`npm run export`を実行すると、TV Showsの各ページが静的コンテンツとして`/out`に出力される。`npm install -g serve`でHTTPサーバをインストールし、`serve`で配信するとBatman TV Showsへアクセスできる。

```jsx
$ npm run export

> hello-next@1.0.0 export /Users/mshr/next-learn-demo/E1-static-export
> next export

> using build directory: /Users/mshr/next-learn-demo/E1-static-export/.next
  copying "static build" directory
  launching 3 workers
[   =] Exporting (1/14)Fetched show: The Batman
Show data fetched. Count: 10
[    ] Exporting (1/14)Fetched show: Batman
[ ===] Exporting (4/14)Fetched show: Batman: The Animated Series
Fetched show: The New Batman Adventures
Fetched show: Batman Beyond
[=   ] Exporting (7/14)Fetched show: Beware the Batman
Fetched show: Batman: Black and White
[  ==] Exporting (9/14)Fetched show: Batman Unlimited
[==  ] Exporting (12/14)Fetched show: The Adventures of Batman
[====] Exporting (13/14)Fetched show: Batman: The Brave and the Bold
Exporting (14/14)

Export successful
mshrs-MacBook-Pro:out mshr$ ls
404             _next           index.html
404.html        about           show
mshrs-MacBook-Pro:out mshr$ serve -p 8080

   ┌────────────────────────────────────────────────────┐
   │                                                    │
   │   Serving!                                         │
   │                                                    │
   │   - Local:            http://localhost:8080        │
   │   - On Your Network:  http://192.168.0.1:8080      │
   │                                                    │
   │   Copied local address to clipboard!               │
   │                                                    │
   └────────────────────────────────────────────────────┘
```

# TypeScript

Next.jsはTypeScriptに対応している。このレッスンでは、新しくプロジェクトを作り、その中で作業を進める。

```bash
mkdir next-ts
cd next-ts
npm init -y
npm install --save react react-dom next
npm install --save-dev typescript @types/react @types/node
```

`package.json`を開き、`script`を以下に書き換える。

```bash
"scripts": {
  "dev": "next",
  "build": "next build",
  "start": "next start"
}
```

`pages/index.tsx`を作り、以下を記述する。

```bash
const Home = () => <h1>Hello world!</h1>;

export default Home;
```

`npm run dev`を実行すると、開発サーバが起動するとともに`tsconfig.json`が自動生成された旨のメッセージが端末に表示される。

次に、`pages/index.tsx`を下記に書き換え、

```bash
const Home = ({ userAgent }) => <h1>Hello world! - user agent: {userAgent}</h1>;

Home.getInitialProps = async ({ req }) => {
  const userAgent = req ? req.headers['user-agent'] : navigator.userAgent;
  return { userAgent };
};

export default Home;
```

`tsconfig.json`内の`strict`を`"strict": true`へ書き換え、[http://localhost:3000](http://localhost:3000/)へアクセスすると、型エラーが表示される。

```tsx
ERROR in /Users/mshr/next-ts/pages/index.tsx(1,17):
1:17 Binding element 'userAgent' implicitly has an 'any' type.
  > 1 | const Home = ({ userAgent }) => <h1>Hello world! - user agent: {userAgent}</h1>;
      |                 ^
    2 | 
    3 | Home.getInitialProps = async ({ req }) => {
    4 |   const userAgent = req ? req.headers['user-agent'] : navigator.userAgent;
```

`index.tsx`を下記に書き換え、型情報を付与することで、エラーが解消される。

```tsx
import { NextPage } from 'next';

const Home: NextPage<{ userAgent: string }> = ({ userAgent }) => (
  <h1>Hello world! - user agent: {userAgent}</h1>
);

Home.getInitialProps = async ({ req }) => {
  const userAgent = req ? req.headers['user-agent'] || '' : navigator.userAgent;
  return { userAgent };
};

export default Home;
```

# Create AMP Pages

アプリケーションをAMP対応させる。`config`オブジェクトの`amp: true`をエクスポートすることで、常にAMP対応ページになる。

```jsx
// pages/index.js
export const config = { amp: true };

export default function Index(props) {
  return <p>Welcome to the AMP only Index page!!</p>;
}
```

`amp: 'hybrid'`とすることで、URLへ`?amp=1`を付けるとAMPページが、付けないと通常のページがレンダリングされる。

```jsx
// pages/index.js
import { useAmp } from 'next/amp';

export const config = { amp: 'hybrid' };

export default function Index(props) {
  const isAmp = useAmp();
  return <p>Welcome to the {isAmp ? 'AMP' : 'normal'} version of the Index page!!</p>;
}
```

# Automatic Static Optimization

Next.js 9以降のバージョンでは、動的な実装を含むページはServerとして、静的な実装のみならStaticとしてビルドしてくれる。

以下の実装を`index.js`として保存する。

```jsx
const Index = () => <h1>Hello World</h1>;
export default Index;
```

`npm run build`でビルドすると、Staticとしてビルドされたことが確認できる。

```jsx
mshrs-MacBook-Pro:hello-next mshr$ npm run build

> hello-next@1.0.0 build /Users/mshr/hello-next
> next build

Creating an optimized production build

Compiled successfully.

Automatically optimizing pages

Page                                                           Size     First Load JS
┌ ○ /                                                          266 B          58.2 kB
└ ○ /404                                                       3.15 kB        61.1 kB
+ First Load JS shared by all                                  57.9 kB
  ├ static/pages/_app.js                                       957 B
  ├ chunks/bd49b44e65de90cbf4f977910e0c706119f124ab.92878c.js  10.3 kB
  ├ chunks/framework.0f140d.js                                 40 kB
  ├ runtime/main.43a0bb.js                                     5.95 kB
  └ runtime/webpack.b65cab.js                                  746 B

λ  (Server)  server-side renders at runtime (uses getInitialProps or getServerSideProps)
○  (Static)  automatically rendered as static HTML (uses no initial props)
●  (SSG)     automatically generated as static HTML + JSON (uses getStaticProps)
```

次に、`agent.js`として以下を保存し、

```jsx
const Agent = ({ userAgent }) => <h1>Your user agent is: {userAgent}</h1>;

Agent.getInitialProps = async ({ req }) => {
  const userAgent = req ? req.headers['user-agent'] : navigator.userAgent;
  return { userAgent };
};

export default Agent;
```

同様に`npm run build`でビルドすると、今度はServerとしてビルドされる。`getInitialProps`を含むため、動的ページとして認識されている。

```jsx
mshrs-MacBook-Pro:.next mshr$ npm run build

> hello-next@1.0.0 build /Users/mshr/hello-next
> next build

Creating an optimized production build

Compiled successfully.

Automatically optimizing pages

Page                                                           Size     First Load JS
┌ ○ /                                                          266 B          58.6 kB
├ ○ /404                                                       3.15 kB        61.4 kB
└ λ /agent                                                     400 B          58.7 kB
+ First Load JS shared by all                                  58.3 kB
  ├ static/pages/_app.js                                       959 B
  ├ chunks/f7db141d5c375743d3f7fa783807f86c90c92554.a3bfbe.js  2.4 kB
  ├ chunks/fbdfa5f92fe41f664c609f7a50034ba170386183.79e44c.js  8.28 kB
  ├ chunks/framework.0f140d.js                                 40 kB
  ├ runtime/main.9a37d4.js                                     5.95 kB
  └ runtime/webpack.b65cab.js                                  746 B

λ  (Server)  server-side renders at runtime (uses getInitialProps or getServerSideProps)
○  (Static)  automatically rendered as static HTML (uses no initial props)
●  (SSG)     automatically generated as static HTML + JSON (uses getStaticProps)
```

# まとめ

Next.js Learn (Excel)では、静的コンテンツとしてエクスポートする方法、TypeScript、アプリケーションをAMP対応する方法、Automatic Static Optimizationによるページの自動静的/動的ビルドを学んだ。今回学んだことをベースに、このブログをカスタマイズしていきたい。
