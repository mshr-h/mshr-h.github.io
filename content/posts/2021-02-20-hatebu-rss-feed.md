---
title: "はてブ検索結果をRSSで取得する"
slug: "hatebu-rss-feed"
subtitle:    ""
description: ""
date:        "2021-02-20"
author:      "Masahiro Hiramori"
tags:        ["tips"]
categories:  ["Tech"]
draft:       false
---

はてブの検索結果をRSSリーダーで取得したい場合、検索結果のURLに`&mode=rss`を追加したURLを登録する。

例えば、キーワード「pybind11」の検索結果URLは以下。

- `https://b.hatena.ne.jp/search/text?q=pybind11`

この検索結果をRSSリーダーで読みたい場合、`&mode=rss`を追加した以下のURLを登録する。

- `https://b.hatena.ne.jp/search/text?q=pybind11&mode=rss`
