---
title: "yaml_cvによる履歴書PDFのビルドをGitHub Actionsでやる"
slug: "resume-pdf-github-actions"
subtitle:    ""
description: ""
date:        "2020-05-06"
author:      "Masahiro Hiramori"
image:       ""
tags:        ["GitHub"]
categories:  ["Tech" ]
draft:       false
---

履歴書PDFのビルド作業をGitHub Actionsで定義した。
リポジトリ自体は[yaml_cv](https://github.com/kaityo256/yaml_cv)をクローンし、プライベートリポジトリとして作成した。
以下の内容を`.github/workflows/create_pdf.yaml`として保存し、リポジトリにプッシュするだけ。

```yaml
name: Create PDF
on: push

jobs:
    lint:
        name: Create PDF
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v2
            with:
              fetch-depth: 1
          - uses: actions/setup-ruby@v1
            with:
              node-version: "2.7"
          - run: |
              gem install prawn
              ruby make_cv.rb -i data.yaml -s style.txt -o output.pdf
          - uses: actions/upload-artifact@v1
            with:
              name: create-pdf
              path: ./output.pdf
```

ビルドに成功すると以下のようにActionsタブにビルド結果が表示される。`create-pdf`をクリックするとPDFがダウンロードできる。

{{< figure src="/img/post/2020-05-06-github-actions.png" width="75%" height="75%" >}}
