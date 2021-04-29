# TVM VTAに関するメモ書き


TVM VTAに関してのメモ書き。2021/4/29現在の情報。

## Q. どのボードを選ぶべきか

A. PYNQ-Z1がおすすめ

現時点VTAのInstallページには、PYNQ-Z1とDE10-Nanoのセットアップ手順が書かれている。

しかしDE10-Nanoは手元で試した限り、LinuxのCMA(Contiguous Memory Allocation)周りが正しく設定されていないようで動かなかった。
TVM Community Discussionを見ても動かない旨の報告がある。

一方PYNQ-Z1は、セットアップ手順通りに進めれば動いた。
VTAをとりあえず試してみたいという目的であれば、PYNQ-Z1がおすすめ。

## mainブランチで動かない可能性がある

最近VTA周りの変更が多く、かつVTAのテストケースが貧弱のため、mainブランチでVTAが動かない事がよくある。
手順通りに進めてもエラーが出るのであれば、古いコミットを試すとよい。

