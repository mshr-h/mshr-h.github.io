# Raspberry Pi 4上のUbuntu 21.04でFirecrackerを動かしてみた


手順メモ。
公式のInstructionはここ。

- [firecracker/getting-started.md at main · firecracker-microvm/firecracker](https://github.com/firecracker-microvm/firecracker/blob/main/docs/getting-started.md)

KVMは有効済みなのでOK。

## 拡張ACL設定

`/dev/kvm`にアクセスするための設定。

```bash
sudo apt install acl
sudo setfacl -m u:${USER}:rw /dev/kvm
```

## Firecrackerバイナリ取得

ダウンロード。

```bash
release_url="https://github.com/firecracker-microvm/firecracker/releases"
latest=$(basename $(curl -fsSLI -o /dev/null -w  %{url_effective} ${release_url}/latest))
arch=`uname -m`
curl -L ${release_url}/download/${latest}/firecracker-${latest}-${arch}.tgz \
| tar -xz
```

バイナリを`/usr/local/bin/`に移動。

```bash
cd release-${latest}
mv firecracker-${latest}-${arch} firecracker
chmod +x firecracker
sudo mv firecracker /usr/local/bin/
```

## Firecracker実行

Firecrackerプロセスを起動し、API呼び出しを受け付ける準備をする。

```bash
sudo rm -f /tmp/firecracker.socket
firecracker --api-sock /tmp/firecracker.socket
```

以降は別の端末を起動して実行。

## MicroVM起動

以下のスクリプトを実行してKernelとRootFSを取得。

```
arch=`uname -m`
dest_kernel="hello-vmlinux.bin"
dest_rootfs="hello-rootfs.ext4"
image_bucket_url="https://s3.amazonaws.com/spec.ccfc.min/img"

if [ ${arch} = "x86_64" ]; then
    kernel="${image_bucket_url}/quickstart_guide/x86_64/kernels/vmlinux.bin"
    rootfs="${image_bucket_url}/hello/fsfiles/hello-rootfs.ext4"
elif [ ${arch} = "aarch64" ]; then
    kernel="${image_bucket_url}/quickstart_guide/aarch64/kernels/vmlinux.bin"
    rootfs="${image_bucket_url}/aarch64/ubuntu_with_ssh/fsfiles/xenial.rootfs.ext4"
else
    echo "Cannot run firecracker on $arch architecture!"
    exit 1
fi

echo "Downloading $kernel..."
curl -fsSL -o $dest_kernel $kernel

echo "Downloading $rootfs..."
curl -fsSL -o $dest_rootfs $rootfs

echo "Saved kernel file to $dest_kernel and root block device to $dest_rootfs."
```

以下のスクリプトを実行して、MicroVMに対してKernelを設定する。

```bash
arch=`uname -m`
kernel_path=$(pwd)"/hello-vmlinux.bin"

if [ ${arch} = "x86_64" ]; then
    curl --unix-socket /tmp/firecracker.socket -i \
      -X PUT 'http://localhost/boot-source'   \
      -H 'Accept: application/json'           \
      -H 'Content-Type: application/json'     \
      -d "{
            \"kernel_image_path\": \"${kernel_path}\",
            \"boot_args\": \"console=ttyS0 reboot=k panic=1 pci=off\"
       }"
elif [ ${arch} = "aarch64" ]; then
    curl --unix-socket /tmp/firecracker.socket -i \
      -X PUT 'http://localhost/boot-source'   \
      -H 'Accept: application/json'           \
      -H 'Content-Type: application/json'     \
      -d "{
            \"kernel_image_path\": \"${kernel_path}\",
            \"boot_args\": \"keep_bootcon console=ttyS0 reboot=k panic=1 pci=off\"
       }"
else
    echo "Cannot run firecracker on $arch architecture!"
    exit 1
fi
```

実行すると以下が出力される。

```bash
HTTP/1.1 204
Server: Firecracker API
Connection: keep-alive
```

同様に以下のスクリプトを実行して、MicroVMに対してRootFSを設定する。

```bash
rootfs_path=$(pwd)"/hello-rootfs.ext4"
curl --unix-socket /tmp/firecracker.socket -i \
  -X PUT 'http://localhost/drives/rootfs' \
  -H 'Accept: application/json'           \
  -H 'Content-Type: application/json'     \
  -d "{
        \"drive_id\": \"rootfs\",
        \"path_on_host\": \"${rootfs_path}\",
        \"is_root_device\": true,
        \"is_read_only\": false
   }"
```

実行すると以下が出力される。

```bash
HTTP/1.1 204
Server: Firecracker API
Connection: keep-alive
```

MicroVMを起動する。

```bash
curl --unix-socket /tmp/firecracker.socket -i \
    -X PUT 'http://localhost/actions'       \
    -H  'Accept: application/json'          \
    -H  'Content-Type: application/json'    \
    -d '{
        "action_type": "InstanceStart"
     }'
```

コマンドを実行した端末で以下が出力され、

```bash
HTTP/1.1 204
Server: Firecracker API
Connection: keep-alive
```

Firecrackerプロセスを起動した端末でMicroVMが起動する。
ユーザ名、パスワードともに`root`でログインできる。

```
[  OK  ] Started Serial Getty on ttyS0.
[  OK  ] Reached target Login Prompts.
[  OK  ] Started OpenBSD Secure Shell server.
[  OK  ] Reached target Multi-User System.
[  OK  ] Reached target Graphical Interface.
         Starting Update UTMP about System Runlevel Changes...
[  OK  ] Started Update UTMP about System Runlevel Changes.

Ubuntu 18.04.2 LTS fadfdd4af58a ttyS0

fadfdd4af58a login:
```

## 感想

- firecrackerプロセスの起動時のメモリ使用量は924KB程度
    - 軽い。。。
- 起動時間は計ってないが高速
    - curlコマンドを実行したらすぐにMicroVMが起動する

