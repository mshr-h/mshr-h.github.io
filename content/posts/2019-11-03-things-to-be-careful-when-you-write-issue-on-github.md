---
title: "OSSメンテナをしていて気づいた、Issue報告時に気をつけて欲しいこと"
slug: "things-to-be-careful-when-you-write-issue-on-github"
subtitle:    ""
description: ""
date:        "2019-11-03"
author:      "Masahiro Hiramori"
tags:        []
categories:  ["Tech" ]
draft:       false
---

私がメンテナであるOSS([Verilog HDL/SystemVerilog向けのVS Codeプラグイン](https://marketplace.visualstudio.com/items/mshr-h.VerilogHDL))のIssueを見ていて気づいた、Issue報告時に気をつけて欲しいことをつらつら書いた。
OSSの規模感はインストール:9.2万、Star:63、Fork:26。

---

# 再現手順を記載する

「〇〇が正しく動かない」とだけ書かれたIssueがたまにある。OSやソフトウェアバージョン等の実行環境、再現手順、期待される結果が書かれていると、こちらで調査しやすくなる。

# 1つのIssueには1つの問題だけ記載する

「〇〇という機能と××という機能が動かない」のようなIssueもたまにある。複数の問題を1つのIssueに詰め込まれると、手を付けるのが面倒になって後回しになることが多々あるので、1つのIssueには1つの問題だけ記載してほしい。

# おわりに

とまあ、気をつけてほしいことを書いたが、以下のテンプレートを参考にIssue報告を書けば問題ないかと思う。

[Github Issue Template Example - Loki Docs](https://lokidocs.com/Contributing/Issue_Template/)