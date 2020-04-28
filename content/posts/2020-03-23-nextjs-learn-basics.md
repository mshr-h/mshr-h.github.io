---
title: "Next.js Learn (Basics)でNext.jsの基礎を学んだ"
slug: "nextjs-learn-basics"
subtitle:    ""
description: ""
date:        "2020-03-23"
author:      "Masahiro Hiramori"
tags:        ["Next.js"]
categories:  ["Tech" ]
draft:       false
---

Next.jsの公式チュートリアル[Next.js Learn (Basics)](https://nextjs.org/learn/basics/getting-started)でNext.jsの基礎を学んだ。

# 背景

本ブログは、Next.js+Notionでブログを作れる[Notion Blog](https://github.com/ijjk/notion-blog)をベースに構築している。React、nodejs、Next.js等のWebフロントエンドは未経験者なので、ブログのカスタマイズに必要な知識の獲得を目的にチュートリアルを実施した。

# 内容

チュートリアルは9つのレッスンで構成されている。全部実施するとおよそ3時間程度で終わる。

- [Getting Started](https://nextjs.org/learn/basics/getting-started)
- [Navigate Between Pages](https://nextjs.org/learn/basics/navigate-between-pages)
- [Using Shared Components](https://nextjs.org/learn/basics/using-shared-components)
- [Create Dynamic Pages](https://nextjs.org/learn/basics/create-dynamic-pages)
- [Clean URLs with Dynamic Routing](https://nextjs.org/learn/basics/clean-urls-with-dynamic-routing)
- [Fetching Data for Pages](https://nextjs.org/learn/basics/fetching-data-for-pages)
- [Styling Components](https://nextjs.org/learn/basics/styling-components)
- [API Routes](https://nextjs.org/learn/basics/api-routes)
- [Deploying a Next.js App](https://nextjs.org/learn/basics/deploying-a-nextjs-app)

各レッスンは繋がっており、全部実施すると計4つのアプリを作ることになる。

- Hello World
    - プロジェクト作成、サイト内遷移、共有コンポーネントを学ぶ
- Blog
    - 動的ページ、動的ルーティング、styled-jsx、Nowへのデプロイ方法を学ぶ
- TV Shows
    - ページデータのフェッチを学ぶ
- Famous Quotes
    - APIの作り方、使い方を学ぶ

下記リポジトリに各レッスンの開始状態が公開されているため、最初から実施する必要はなく途中からでも可能。

- [zeit/next-learn-demo](https://github.com/zeit/next-learn-demo)

# 作成したアプリ(Famous Quotes)

{{< figure src="/img/post/2020-03-23-nextjs-learn-basic.png" class="center" width="50%" height="50%" >}}

# 学習メモ

- `pages/p/[id].tsx`のファイル名の`[id]`はdynamic route機能を意味する
- `export default () =>...`でReact Componentをエクスポートする
- `Link`でページ内遷移する
- `getInitialProps`を使い初回ロード時に実行する処理を記述する