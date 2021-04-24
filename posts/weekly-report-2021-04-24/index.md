# Weekly Report(2021/04/24)


Look back on the week.

## English

- DMM英会話のグレードがシルバーになった
    - 最初の5回ぐらいで頭打ちになった感がある
- 英会話でApple新製品関連の話をした
    - 自分がよく知っている話題のため、かなり話しやすいと感じた

## Machine learning / Cloud computing

- 各種評価ボードを購入
    - STM Nucleo-F746ZG
    - PYNQ-Z1
- PYNQ-Z1でTVM動かした
    - `test_benchmark_topi_conv2d.py`の動作OK
    - `resnet18_v1`を動かすと、ARM CPUのみだと6540.10ms、VTA使うと819.5msと、約8倍高速
    - いろいろモデル動かしてみたい
- TVMの[Caffe frontend](https://discuss.tvm.apache.org/t/introduce-new-frontend-for-caffe/6918)試した
    - ResNet-18、CaffeModelは動いたが、SSD系は対応していない

## Rock climbing

- MoonBoardのV5~V7あたりを触るなど
- Climbing World CupのMeiringen大会が開催
    - [IFSC Boulder World Cup Meiringen 2021 || Men's and women's semi-finals - YouTube](https://www.youtube.com/watch?v=SA2K2o2kCCQ)
    - [IFSC Boulder World Cup Meiringen 2021 || Men's and women's finals - YouTube](https://www.youtube.com/watch?v=sO7KPBUeDWY)

## Other topics

- Obsidian使い始めて約10日経過
    - daily noteに全部書く→ある程度情報が溜まったらpageに分割、という流れでnote-takingしている
    - tagとpageをごちゃまぜに使ってるのは良くない気がする
        - 現状はLinuxというpageと\#linuxというtagが存在してたりする
- カフェイン減らす生活を始めた
- [Software 2.0とその社会的課題](https://tech.preferred.jp/ja/blog/software2_0/)
    - 読み物
- [Copy Title and Url as Markdown Style - Chrome Web Store](https://chrome.google.com/webstore/detail/copy-title-and-url-as-mar/fpmbiocnfbjpajgeaicmnjnnokmkehil?hl=en)
    - WebページのタイトルとURLをMarkdown形式でコピーするChrome拡張
    - Obsidianのdaily noteに貼り付ける時に利用
    - キーボードショートカットは`Ctrl+Shift+C`に割当て
- [Machine Learning Datasets | Papers With Code](https://paperswithcode.com/datasets)
    - Papers With CodeでDatasetを検索できるようになった
- [Flashlight](https://github.com/flashlight/flashlight)
    - C++で書かれた機械学習ライブラリ

