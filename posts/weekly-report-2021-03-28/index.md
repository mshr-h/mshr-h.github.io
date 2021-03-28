# Weekly Report(2021/03/28)


今週の振り返り。

## 英語

- 今週は4回レッスンを受けた
  - 忙しくて毎日受けれず
- 会話トピックは先週と変わらず、趣味のロッククライミング、ゲーム、英語を勉強している理由など

## Machine Learning

- TVM VTAを触り始めた
  - DE10Nanoの環境構築したが、`test_benchmark_topi_conv2d.py`が途中でエラー吐く
- Gradient free optimizer
  - https://github.com/SimonBlanke/Gradient-Free-Optimizers
  - 微分フリー最適化のPythonライブラリ
  - 使いやすそう

## Rock climbing

- ホームジムのV6を触るなど
- 外岩も行きたい
  - 4月中に何回か行きたい

## その他トピック

- [Docker for M1 Mac RC](https://docs.docker.com/docker-for-mac/apple-m1/)
- Architecture Decision Records(ADR)
  - ソフトウェアアーキテクチャの重要な決定を背景・結果とともに記録する手法
  - いくつかやり方があるみたいだが、共通しているのは「バージョン管理する」、「ソースコードとともに管理する(`adr/`のようなディレクトリに保管)」、「1つの決定ごとに対して1つのファイル」
  - [ADR Tools](https://github.com/npryce/adr-tools)
    - ADRのためのコマンドラインツール
- ブログに[Search]({{< ref "/search" >}})を追加した
  - Hugoのビルド時に[全記事のインデックス](../../index.json)を作成し、検索時にインデックス内の記事1つ1つに対して検索ワードで`String.prototype.match()`する単純なやり方
  - 記事数が少ないので、このやり方問題ない
  - 検索結果の表示をもうちょっと見やすくしたい


