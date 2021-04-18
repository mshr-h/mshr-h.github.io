# Weekly Report(2021/04/18)


Look back on the week.

## English

- 今週も毎日受けた
  - しかしあんまり成長を感じれてない

## Machine learning / Cloud computing

- DE10-Nano に VTA 環境を構築しているが苦戦中
  - Angstrom Linux のカーネルと dtb を変更、cma.ko をロードしたら`/dev/cma`が生えるところまでは確認できた
  - VTA のチュートリアルを動かすと`cma_free`でエラーが発生する
  - CMA(Contiguous Memory Allocator)のメモリ解放の挙動がおかしい？
- [Cerebras の新しい挑戦 ――データフローマシンとして流体力学問題を解く](https://gihyo.jp/dev/column/01/ml/2021/cerebras)
  - Cerebras の WSE(Wafer Scale Engine)を計算流体力学に応用した事例
  - 隣接するコア間の通信メッセージに 24 種類のタグを付けられるとあるが、なんで中途半端な数字なんだろうか
    - $$2^n$$の方が都合がいいと思う

## Rock climbing

- あんまり登れず
- 週末のボルダリングワールドカップが楽しみ

## Other topics

- Obsidian
  - Web clip以外のNotionデータを移行した
  - Daily notes設定してみたので、しばらく運用する
- [WSL 2 に特化した「Ubuntu on Windows Community Preview」とは - 阿久津良和の Windows Weekly Report](https://news.mynavi.jp/article/20210411-windows10report/)
  - [[Ubuntu]] の Preview 版的な立ち位置
  - とりあえず入れていろいろ触ってみる
- HPVM: Heterogeneous Parallel Virtual Machine
  - https://dl.acm.org/doi/pdf/10.1145/3200691.3178493
  - https://hpvm.readthedocs.io/en/latest/index.html

