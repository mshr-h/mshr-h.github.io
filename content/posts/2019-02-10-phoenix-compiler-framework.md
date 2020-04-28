---
title: "Visual C++コンパイラとPhoenix Compiler Framework"
slug: "phoenix-compiler-framework"
subtitle:    ""
description: ""
date:        "2019-02-10"
author:      "Masahiro Hiramori"
tags:        []
categories:  ["Tech" ]
draft:       false
---

Visual C++コンパイラについて調べたのでメモ。

# Visual C++コンパイラとは

Microsoft Visual C++という、Microsoft製のC/C++統合開発環境に付属のC/C++コンパイラ。Visual Studioをインストールする際にC/C++開発サポートを有効にすると、Microsoft Visual C++が導入される。このVisual C++コンパイラ自体は、標準のインストール先であれば`C:\\Program Files (x86)\\Microsoft Visual Studio 14.0\\VC\\bin`に存在し、以下の実行ファイルとDLLで構成される。

- `cl.exe`：コンパイラのドライバ
- `c1.dll`：Visual Cのフロントエンド
- `c1xx.dll`：Visual C++のフロントエンド
- `c2.dll`：バックエンド

![/img/post/2019-02-10-phoenix-compiler.png](/img/post/2019-02-10-phoenix-compiler.png)

`cl.exe`

コンパイラのドライバで、フロントエンド、バックエンドをつなぐ役割。

`c1.dll`、`c1xx.dll`

コンパイラのフロントエンド処理を実行する。C/C++ソースコードを解析し、MSIL(Microsoft Intermediate Language)形式の中間言語を生成する。ソースコードがC言語の場合は`c1.dll`、C++言語の場合は`c1xx.dll`により処理される。

`c2.dll`

コンパイラのバックエンド処理を実行する。フロントエンドで生成されたMSILを受取り、最適化、コード生成(オブジェクトファイル生成)を行う。

# Phoenix Compiler Frameworkとは

Microsoftが2003年頃？から研究開発しているコンパイラフレームワーク。プラグイン機構を採用することで、コンパイラのフロントエンド、バックエンドを取り替えることができる。これにより、新たなプログラミング言語/CPUアーキテクチャへの対応に必要な開発コストを削減可能となる。Visual C++コンパイラはこのPhoronix Compiler Frameworkがベースになってるよう。

# 余談

LLVMの作者の一人であるChris Lattnerは学生時代、Microsoft ResearchのインターンでPhoenix Compiler Frameworkの開発に関わっていたらしい。具体的には、MicrosoftコンパイラとLLVMコンパイラのブリッジ(LLVMが.NETコードをコンパイル可能とする)の開発を行っていたそう。

# 参考文献

- [Phoenix (compiler framework) — Wikipedia](https://en.wikipedia.org/wiki/Phoenix_(compiler_framework))
- [An Explanation of the Phoenix Compiler Framework](https://www.infoq.com/news/2008/05/Phoenix-Compiler-Framework)
- [Andy Ayers: Understanding the Phoenix Compiler Framework](https://channel9.msdn.com/Shows/Going+Deep/Andy-Ayers-Understanding-the-Phoenix-Compiler-Framework)
- [CppCon 2015: James Radigan “CLANG + C2 — Engineering/Futures/Measurements” — Youtube](https://www.youtube.com/watch?v=TRgWJuQhkQo)
- [Phoenix: Experience with an Analysis and Optimization Framework — Microsoft Research](https://www.microsoft.com/en-us/research/video/phoenix-experience-with-an-analysis-and-optimization-framework/)
- [From Mole Hills to Mountains: Revealing Rich Header and Malware Triage](From Mole Hills to Mountains: Revealing Rich Header and Malware Triage)
- [Chris Lattner’s Resume](http://nondot.org/sabre/Resume.html)