---
title:       "Ubuntu 18.04 LTSでibus-mozcのキーボード配列が日本語にならない問題と解決方法"
subtitle:    ""
description: ""
date:        2018-06-04
author:      "Masahiro Hiramori"
tags:        ["Linux"]
categories:  ["Tech" ]
draft:       false
---

Ubuntu 18.04にibus-mozcをインストールしたところ、キーボード配列が日本語にならなかった。ibus-mozcの設定を書き換えると解決したので防備録。

## 検証環境

- Ubuntu 18.04 LTS
- ibus-mozc 2.20.2673.102

## 解決策

ibus-mozcの設定ファイルを開く。

```bash
sudo vi /usr/share/ibus/component/mozc.xml
```

下から6行目の<layout>default</layout>を<layout>jp</layout>に書き換える。

```xml
<component>
  <version>2.20.2673.102+dfsg-2</version>
  <name>com.google.IBus.Mozc</name>
  <license>New BSD</license>
  <exec>/usr/lib/ibus-mozc/ibus-engine-mozc --ibus</exec>
  <textdomain>ibus-mozc</textdomain>
  <author>Google Inc.</author>
  <homepage><https://github.com/google/mozc></homepage>
  <description>Mozc Component</description>
<engines>
<engine>
  <description>Mozc (Japanese Input Method)</description>
  <language>ja</language>
  <symbol>&#x3042;</symbol>
  <rank>80</rank>
  <icon_prop_key>InputMode</icon_prop_key>
  <icon>/usr/share/ibus-mozc/product_icon.png</icon>
  <setup>/usr/lib/mozc/mozc_tool --mode=config_dialog</setup>
  <layout>default</layout>
  <name>mozc-jp</name>
  <longname>Mozc</longname>
</engine>
</engines>
</component>
```

ログアウトし、再ログインするとキーボード配列が日本語になる。