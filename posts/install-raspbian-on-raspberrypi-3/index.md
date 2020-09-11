# Raspberry Pi 3にRaspbian Busterを導入する


Raspberry Pi 3 Model BにRaspbian Busterを導入したので作業メモ。

# 実施環境

- MacBook Pro 13インチ 2017年モデル
    - macOS Catalina 10.15.4
- Raspberry Pi 3 Model B
- Raspbian Buster with desktop

# Raspbianイメージを取得

```bash
$ cd ~/Downloads
$ wget http://downloads.raspberrypi.org/raspbian/images/raspbian-2020-02-14/2020-02-13-raspbian-buster.zip
```

# Raspbianイメージを書き込み

microSDカードをMacBook Proに接続する。以下のコマンドでイメージを書き込む。

```bash
$ diskutil list
$ diskutil umountDisk /dev/disk2
$ cd ~/Downlaods
$ unzip 2020-02-13-raspbian-buster.zip
$ sudo dd bs=1m if=2020-02-13-raspbian-buster.img of=/dev/rdisk2 conv=sync
$ sync
$ diskutil umountDisk /dev/disk2
```

Raspberry Piにマウス、キーボード、ディスプレイを接続する。

microSDをRaspberry Piに入れ、microUSBケーブルを接続すると起動。

# 初期設定

デスクトップが起動すると初期設定画面が表示されるので、ユーザパスワード、無線LAN接続などを設定する。

デスクトップが表示されたら、端末から`sudo raspi-config`コマンドでコンフィグを起動する。

`Interfacing Options`を選択し、カメラを接続するため`P1 Camera`と、開発用PCからリモートで接続できるように`P2 SSH`と`P3 VNC`をそれぞれ有効にする。

`ifconfig`コマンドでIPアドレスを確認し、再起動する。開発用PCからSSHでRaspberry Piの`pi`ユーザへ接続できれば設定完了。

# 参考文献

- [https://www.raspberrypi.org/downloads/raspbian/](https://www.raspberrypi.org/downloads/raspbian/)
- [https://www.raspberrypi.org/documentation/installation/installing-images/mac.md](https://www.raspberrypi.org/documentation/installation/installing-images/mac.md)
