# Weekly Report(2021/05/22)


Look back on the week.

## English

- 6日受けた
    - bucketlist、TED、rock climbingの話をした

## Machine learning / Cloud computing

- microTVMのstandaloneアプリを作ってる
    - [nucleo-f746zg-microtvm-example/standalone at main · mshr-h/nucleo-f746zg-microtvm-example](https://github.com/mshr-h/nucleo-f746zg-microtvm-example/tree/main/standalone)
    - とりあえずコードをコピーしただけ
    - runtimeとモデルをCコードに変換した結果を用意すればよさそう
- Raspberry Pi 4にUbuntu 21.04とDocker、[balenaEngine](https://www.balena.io/engine/)、Apache TVMを入れて遊んでる
    - TVMをビルドする際、LLVM-10、LLVM-11だと`libPolly.a`が見つからない旨のエラーが出るため、LLVM-9を入れた

## Rock climbing

- Moonboardは触らず
- ホームジムのWeekly課題を登った

## Other topics

- [How to disable/enable GUI on boot in Ubuntu 20.04 Focal Fossa Linux Desktop - LinuxConfig.org](https://linuxconfig.org/how-to-disable-enable-gui-on-boot-in-ubuntu-20-04-focal-fossa-linux-desktop)
- [git bisect](https://git-scm.com/docs/git-bisect)
    - 存在は知っていたが、使ったことなかった
    - 便利そうなので今度使う
- [エンジニアの楽園 vim-jp | 日々、とんは語る。](https://blog.tomoya.dev/posts/vim-jp-is-a-paradise-for-engineers/)
    - 入った
    - Rock climbing channelがなさそうなので、やってる人が多そうなら作りたい
- [Raspberry Pi4のGPGPUに挑戦（その１） - あざらしなので](https://taiki-azrs.hatenablog.com/entry/2021/05/18/190052)
    - Apache TVMにRaspberry Pi 4 GPUサポートを追加するのも面白そう
- [コンウェイの法則](https://ja.wikipedia.org/wiki/%E3%83%A1%E3%83%AB%E3%83%B4%E3%82%A3%E3%83%B3%E3%83%BB%E3%82%B3%E3%83%B3%E3%82%A6%E3%82%A7%E3%82%A4)
    - システムを設計する組織は、その構造をそっくりまねた構造の設計を生み出してしまう
    - 原文: "Organizations which design systems are constrained to produce designs which are copies of the communication structures of these organizations."
    - 社内を見ててもそんな感じがする
- ジェンガ式プロジェクト管理法
    - "あの手この手で中抜きし、最後に手を出した人に全責任を擦り付けるやり方"
    - 出典は見つけられず

