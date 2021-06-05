# Weekly Report(2021/06/05)


Look back on the week.

## English

- 今週は7/7回受けた
    - 急に話せるようになった気がする
- TutorごとにObsidianでページを作り、得意な話題/盛り上がるtopicをメモする
    - 事前のtopic検討に活用
- 発音練習アプリ[ELSA](https://elsaspeak.com/en/)使い始めた
    - 英会話レッスンで、発音を練習した方がいいと指摘されたので
    - 3ヶ月更新のProを契約
    - thの発音が苦手
        - math, pathすら正しく言えない
- be past participleとbe being+past participleの違い
    - どっちもそんなに変わらない
    - be being+past participleは、話している時点も起こっていることを強調している
    - Gorillas are dangerously stressed by tourists
    - Gorillas are being dangerously stressed by tourists
- 用語
    - big picture
        - 全体像
        - ex) I'm gonna give you a big picture of Machine learning optimization.
    - Practice makes perfect
        - 継続は力なり

## Machine learning / Cloud computing

- microTVMのAOTコンパイルを試してるが動かない
    - サンプルをコンパイルすると、internalディレクトリ下のヘッダファイルがない云々のエラーが出る
- Raspberry Pi 4につないだWebCameraをHTTPで配信するアプリを作った
    - [mshr-h/video_streaming: Simple video streaming server written in Python with Redis and Tornado](https://github.com/mshr-h/video_streaming)
    - 1コアフル使用で30 fpsぐらい出てる
    - 次はカメラ映像に対してObject detectionやClassificationをやりたい

## Rock climbing

- ホームジムのweeklyを触るなど
- 今週はMoonboardを触らず

## Other topics

- Flutter
    - [Flutter - Beautiful native apps in record time](https://flutter.dev/)
    - Raspberry Pi向けがあるので、手持ちのRaspberry Pi 4で動かしたい
        - [ardera/flutter-pi: A light-weight Flutter Engine Embedder for Raspberry Pi that runs without X.](https://github.com/ardera/flutter-pi)
        - [flutter-pi を試す - Qiita](https://qiita.com/nanbuwks/items/bd30c7943094e7368a9b)
- [The CFU Playground: Accelerate ML models on FPGAs — CFU-Playground documentation](https://cfu-playground.readthedocs.io/en/latest/)
    - FPGAアクセラレータを開発するためのツールセットのようなもの
    - 面白そうだが、Xilinx Artix 7 35Tを持ってないので試せず
- [Windows 10ミニTips(378) 困った！「別のプログラムがこのフォルダーまたはファイルを開いているので、操作を完了できません」 | マイナビニュース](https://news.mynavi.jp/article/win10tips-378/)
    - 発生するたびに対処方法を検索してるので、いい加減覚える
- TCP 80番ポートを開けるにはroot権限が必要
    - iptablesで8080<->80にリダイレクトするのが安全なやり方
        - [iptables - Redirect port 80 to 8080 and make it work on local machine - Ask Ubuntu](https://askubuntu.com/questions/444729/redirect-port-80-to-8080-and-make-it-work-on-local-machine)

