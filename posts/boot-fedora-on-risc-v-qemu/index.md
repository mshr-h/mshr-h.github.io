# macOSでRISC-V版FedoraをQEMUで起動する


macOS上のQEMUでRISC-V版Fedoraを動かしてみた。ビルド済みバイナリは下記から取得できるものを使用。
- [https://fedorapeople.org/groups/risc-v/disk-images/](https://fedorapeople.org/groups/risc-v/disk-images/)

# 実施環境

- macOS Catalina 10.15.4
- brew導入済み

# 手順

HomebrewでQEMUを導入する。

```sh
$ brew install qemu
```

ワークスペースに移動し、Linuxカーネル、ブートローダ、Fedoraディスクイメージをダウンロードする。

```sh
$ cd ~/workspace
$ mkdir risc-v-fedora
$ wget https://fedorapeople.org/groups/risc-v/disk-images/vmlinux
$ wget https://fedorapeople.org/groups/risc-v/disk-images/bbl
$ wget https://fedorapeople.org/groups/risc-v/disk-images/stage4-disk.img.xz
$ xzdec -d stage4-disk.img.xz > stage4-disk.img
```

以下のコマンドでQEMUを起動する。

```sh
$ qemu-system-riscv64   \
    -nographic \
    -machine virt \
    -kernel bbl \
    -object rng-random,filename=/dev/urandom,id=rng0 \
    -device virtio-rng-device,rng=rng0 \
    -append "console=ttyS0 ro root=/dev/vda" \
    -device virtio-blk-device,drive=hd0 \
    -drive file=./stage4-disk.img,format=raw,id=hd0 \
    -device virtio-net-device,netdev=usernet \
    -netdev user,id=usernet,hostfwd=tcp::10000-:22
```

ユーザ名は`root`、パスワードは`riscv`でログインする。

{{< figure src="/img/post/2020-04-29-fedora-qemu.png" width="75%" height="75%" >}}

ホストのmacOSからSSHで接続もできる。

```
$ ssh -p 10000 root@localhost
```

