---
title: "DNSレコードの種類"
slug: "dns-record"
subtitle:    ""
description: ""
date:        "2020-05-02"
author:      "Masahiro Hiramori"
image:       ""
tags:        []
categories:  ["Tech" ]
draft:       false
---

DNSレコードとは、DNSを動作させるための設定情報のこと。DNSサーバは、ドメイン名とIPアドレスの対応表である「ゾーンファイル」を保持しており、このファイルに記載されている1行ごとの詳細情報をDNSレコードと呼ぶ。
このゾーンファイル設定では、「レコードタイプ」と呼ぶ情報の種類に応じてDNSレコードを記述する。レコードタイプのうち、Vercelで購入したドメインに対して`now dns ls`実行時に表示されるものを次に示す。

- A(Address)レコード
  - ホスト名に対応するIPアドレスを定義する
  - 例えば、独自ドメインでGitHub Pagesを公開する際に使用する
- CNAMEレコード
  - ドメインやホスト名の別の名義を定義する
  - 例えば、www付きドメインからwww無しドメインに転送する際に使用する
- TXTレコード
  - ホスト名に関連付けるテキスト情報を定義する
  - 例えば、Google Search Consoleにドメインプロパティとしてドメインを登録する際に使用する
- MXレコード
  - メールの転送先(メールサーバ)のホスト名を定義する
- CAA(Certification Authority Authorization)レコード
  - SSL/TLSサーバ証明書を発行できる認証局(CA)を指定する
