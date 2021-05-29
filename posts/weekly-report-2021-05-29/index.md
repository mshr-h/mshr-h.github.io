# Weekly Report(2021/05/29)


Look back on the week.

## English

- 5/7日受けた
- 話したいことを事前に準備したほうが上達が早いのは理解しているが、準備が面倒
- おすすめ本、映画、TV seriesは誰でも盛り上がる
- self-improvement、productivity tipsも話が広がる

## Machine learning / Cloud computing

- AWS+Vue.js+Socket.io+NGINX reverse proxyの構成で、Socket.ioが通信できない
    - ローカルでSocket.io+NGINX reverse proxyを構築したら動いたので、AWS特有の問題かも
- Firecracker
    - KVMベースの軽量VM
    - Raspberry Pi 4で試したら、めっちゃ高速に起動して驚いた。
        - [Raspberry Pi 4上のUbuntu 21.04でFirecrackerを動かしてみた]({{< ref "/posts/2021-05-29-firecracker-on-ubuntu-2104-rpi4.md" >}})

## Rock climbing

- 久しぶりにスラブ、垂壁を触る
    - 腹筋を使えてなかったようで、久しぶりに腹筋が筋肉痛になった

## Other topics

- Dockerコンテナ内でGUIアプリを起動する
    - [Lei Mao's Log Book – Docker Container GUI Display](https://leimao.github.io/blog/Docker-Container-GUI-Display/)
    - [How to Run GUI Applications in a Docker Container – CloudSavvy IT](https://www.cloudsavvyit.com/10520/how-to-run-gui-applications-in-a-docker-container/)
    - Dockerのbind mountでX socketをゲストにマウントするのが手っ取り早い
        - セキュリティ的によくないので、コンテナイメージが信用できるときのみ使用する
    - VNC serverを使うやり方もある

