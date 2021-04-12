# DE10-NanoにTVM VTAのRPC Serverを導入する


Terasic [DE10-Nano](https://www.terasic.com.tw/cgi-bin/page/archive.pl?Language=English&No=1046)にTVM VTAのRPC Serverを導入したので作業メモ。

[DE10-Nanoセットアップ](https://tvm.apache.org/docs/vta/install.html#intel-de10-fpga-setup)に従う

## 必要なもの
- DE10-Nano
- microSDカード
  - 8GB以上推奨、microSDXC非対応な気がするので注意
- microUSBケーブル
  - シリアル通信に必要
- LANケーブル
  - ネットワークに繋ぐのに必要

## DE10-Nanoのセットアップ

まずは[Terasicのページ](https://www.terasic.com.tw/cgi-bin/page/archive.pl?Language=English&CategoryNo=167&No=1046&PartNo=4)からAngstrom Linuxイメージをダウンロードする。
- de10-nano-image-Angstrom-v2016.12.socfpga-sdimg.2017.03.31.tgz

ダウンロードしたファイルをmicroSDに書き込む。Win32DiskImagerを使用。

DE10-Nanoに電源ケーブルを接続、microSDをDE10-Nanoに挿入、microUSBケーブルでDE10-NanoとPCを接続する。

PC上でターミナルエミュレータでDE10-Nanoに接続する。TeraTermを使用。

ユーザ名：root、パスワードなしでログインする。

以降はDE10-Nano上で実行する。

### パッケージ導入

システムパッケージをアップデートし、必要なパッケージを導入する。

```sh
opkg update
okpg upgrade
opkg install cmake
opkg install coreutils
```

### Python導入

Angstrom Linuxに入っているPythonは古すぎるので、Pythonソースビルドする。

ソースコードをダウンロードし、ビルド、インストールする。

```sh
mkdir ~/workspace
cd ~/workspace
curl -O https://www.python.org/ftp/python/3.8.8/Python-3.8.8.tgz
tar xvf Python-3.8.8.tgz
cd Python-3.8.8
./configure
make
make install
```

`python`、`python3`コマンドのシンボリックリンクを導入したPythonに置き換える。

```sh
rm /usr/bin/python
ln -s /usr/local/bin/python3.8 /usr/bin/python
rm /usr/bin/python3
ln -s /usr/local/bin/python3.8 /usr/bin/python3
```

`pip`を最新バージョンに更新する。

```sh
/usr/local/bin/python3.8 -m pip install --upgrade pip
```

### Git導入

Angstrom Linuxから導入できるGitはバージョンが古いので、これもソースビルドする。

```sh
opkg install tcl tk gettext
opkg install perl-module-pod-man
cd ~/workspace
git clone https://github.com/git/git
cd git
make configure
./configure --prefix=/usr
make all
make install
```

### TVMのRPC Serverをビルド

TVMの

```sh
cd ~/workspace
git clone --recursive https://github.com/apache/tvm tvm
cd tvm
mkdir build
cp cmake/config.cmake build
echo 'set(USE_VTA_FPGA ON)' >> build/config.cmak
cp 3rdparty/vta-hw/config/de10nano_sample.json 3rdparty/vta-hw/config/vta_config.json
cd build
cmake ..
make clean
make runtime vta -j2
```

TVMのRPC Serverを起動する。

```sh
cd ~/workspace/tvm
./apps/vta_rpc/start_rpc_server.sh
```

ホストPCとの通信に必要になるため、`ip`コマンドなどでDE10-NanoのIPアドレスをメモしておく。

以下のような出力であればOK。

```
INFO:root:RPCServer: bind to 0.0.0.0:9091
```

## 動作確認

RPCサーバーの動作確認をする。以下はホストPCで実行する。

RPC ServerのIPアドレスとポート番号を環境変数に設定する。

```sh
export VTA_RPC_HOST=192.168.10.110
export VTA_RPC_PORT=9091
```

テストスクリプトを実行する。

```sh
cd ~/workspace/tvm
cp 3rdparty/vta-hw/config/de10nano_sample.json 3rdparty/vta-hw/config/vta_config.json
python vta/tests/python/de10nano/test_program_rpc.py
python vta/tests/python/integration/test_benchmark_topi_conv2d.py
```

ホストPCでは以下のような標準出力。途中からエラーを吐いているが、とりあえず動いてそう。

```sh
Conv2DWorkload(batch=1, height=56, width=56, in_filter=64, out_filter=64, hkernel=3, wkernel=3, hpad=1, wpad=1, hstride=1, wstride=1)  
CPU CONV2D TEST PASSED: Time cost = 0.201243 sec/op, 1.14892 GOPS  
Conv2DWorkload(batch=1, height=56, width=56, in_filter=64, out_filter=128, hkernel=3, wkernel=3, hpad=1, wpad=1, hstride=2, wstride=2)  
CPU CONV2D TEST PASSED: Time cost = 0.0930569 sec/op, 1.24231 GOPS  
Conv2DWorkload(batch=1, height=56, width=56, in_filter=64, out_filter=128, hkernel=1, wkernel=1, hpad=0, wpad=0, hstride=2, wstride=2)  
CPU CONV2D TEST PASSED: Time cost = 0.0126302 sec/op, 1.01701 GOPS  
Conv2DWorkload(batch=1, height=28, width=28, in_filter=128, out_filter=128, hkernel=3, wkernel=3, hpad=1, wpad=1, hstride=1, wstride=1)  
CPU CONV2D TEST PASSED: Time cost = 0.194179 sec/op, 1.19071 GOPS  
Conv2DWorkload(batch=1, height=28, width=28, in_filter=128, out_filter=256, hkernel=3, wkernel=3, hpad=1, wpad=1, hstride=2, wstride=2)  
CPU CONV2D TEST PASSED: Time cost = 0.0934884 sec/op, 1.23658 GOPS  
Conv2DWorkload(batch=1, height=28, width=28, in_filter=128, out_filter=256, hkernel=1, wkernel=1, hpad=0, wpad=0, hstride=2, wstride=2)  
CPU CONV2D TEST PASSED: Time cost = 0.0121611 sec/op, 1.05624 GOPS  
Conv2DWorkload(batch=1, height=14, width=14, in_filter=256, out_filter=256, hkernel=3, wkernel=3, hpad=1, wpad=1, hstride=1, wstride=1)  
CPU CONV2D TEST PASSED: Time cost = 0.186729 sec/op, 1.23822 GOPS  
Conv2DWorkload(batch=1, height=14, width=14, in_filter=256, out_filter=512, hkernel=3, wkernel=3, hpad=1, wpad=1, hstride=2, wstride=2)  
CPU CONV2D TEST PASSED: Time cost = 0.11122 sec/op, 1.03943 GOPS  
Conv2DWorkload(batch=1, height=14, width=14, in_filter=256, out_filter=512, hkernel=1, wkernel=1, hpad=0, wpad=0, hstride=2, wstride=2)  
CPU CONV2D TEST PASSED: Time cost = 0.0136669 sec/op, 0.939866 GOPS  
Conv2DWorkload(batch=1, height=7, width=7, in_filter=512, out_filter=512, hkernel=3, wkernel=3, hpad=1, wpad=1, hstride=1, wstride=1)  
CPU CONV2D TEST PASSED: Time cost = 0.223079 sec/op, 1.03645 GOPS  
Conv2DWorkload(batch=1, height=56, width=56, in_filter=64, out_filter=64, hkernel=3, wkernel=3, hpad=1, wpad=1, hstride=1, wstride=1)  
Traceback (most recent call last):  
File "vta/tests/python/integration/test_benchmark_topi_conv2d.py", line 311, in <module>  
test_conv2d(device="vta")  
File "vta/tests/python/integration/test_benchmark_topi_conv2d.py", line 306, in test_conv2d  
vta.testing.run(_run)  
File "/home/ubuntu/workspace/tvm/vta/python/vta/testing/utils.py", line 74, in run  
run_func(env, remote)  
File "vta/tests/python/integration/test_benchmark_topi_conv2d.py", line 304, in _run  
run_conv2d(env, remote, wl, target)  
File "vta/tests/python/integration/test_benchmark_topi_conv2d.py", line 234, in run_conv2d  
data_arr = tvm.nd.array(data_np, ctx)  
File "/home/ubuntu/workspace/tvm/python/tvm/runtime/ndarray.py", line 518, in array  
return empty(arr.shape, arr.dtype, ctx).copyfrom(arr)  
File "/home/ubuntu/workspace/tvm/python/tvm/runtime/ndarray.py", line 292, in empty  
arr = _ffi_api.TVMArrayAllocWithScope(shape_ptr, ndim, dtype, ctx, mem_scope)  
File "tvm/_ffi/_cython/./packed_func.pxi", line 322, in tvm._ffi._cy3.core.PackedFuncBase._call_  
File "tvm/_ffi/_cython/./packed_func.pxi", line 267, in tvm._ffi._cy3.core.FuncCall  
File "tvm/_ffi/_cython/./base.pxi", line 160, in tvm._ffi._cy3.core.CALL  
tvm.error.RPCError: Traceback (most recent call last):  
\[bt\] (8) /home/ubuntu/workspace/tvm/build/libtvm.so(tvm::runtime::NDArray::Empty(std::vector<long, std::allocator<long> >, DLDataType, DLContext, tvm::runtime::Optional<tvm::runtime::String>)+0x124) \[0x7f1a9ab04024\]  
\[bt\] (7) /home/ubuntu/workspace/tvm/build/libtvm.so(tvm::runtime::RPCDeviceAPI::AllocDataSpace(DLContext, int, long const\*, DLDataType, tvm::runtime::Optional<tvm::runtime::String>)+0x89) \[0x7f1a9ab30c59\]  
\[bt\] (6) /home/ubuntu/workspace/tvm/build/libtvm.so(tvm::runtime::RPCClientSession::AllocDataSpace(DLContext, int, long const\*, DLDataType, tvm::runtime::Optional<tvm::runtime::String>)+0x155) \[0x7f1a9ab3ad45\]  
\[bt\] (5) /home/ubuntu/workspace/tvm/build/libtvm.so(+0x164fc14) \[0x7f1a9ab34c14\]  
\[bt\] (4) /home/ubuntu/workspace/tvm/build/libtvm.so(tvm::runtime::RPCEndpoint::HandleUntilReturnEvent(bool, std::function<void (tvm::runtime::TVMArgs)>)+0x37b) \[0x7f1a9ab31a5b\]  
\[bt\] (3) /home/ubuntu/workspace/tvm/build/libtvm.so(tvm::runtime::RPCEndpoint::EventHandler::HandleNextEvent(bool, bool, std::function<void (tvm::runtime::TVMArgs)>)+0x1b8) \[0x7f1a9ab35238\]  
\[bt\] (2) /home/ubuntu/workspace/tvm/build/libtvm.so(tvm::runtime::RPCEndpoint::EventHandler::HandleProcessPacket(std::function<void (tvm::runtime::TVMArgs)>)+0xe7) \[0x7f1a9ab362b7\]  
\[bt\] (1) /home/ubuntu/workspace/tvm/build/libtvm.so(tvm::runtime::RPCEndpoint::EventHandler::HandleReturn(tvm::runtime::RPCCode, std::function<void (tvm::runtime::TVMArgs)>)+0xc9) \[0x7f1a9ab38029\]  
\[bt\] (0) /home/ubuntu/workspace/tvm/build/libtvm.so(dmlc::LogMessageFatal::~LogMessageFatal()+0x6c) \[0x7f1a99beb06c\]
```

DE10-Nanoでは以下のような標準出力。

```sh
INFO:root:Loading VTA library: /home/root/workspace/tvm/vta/python/vta/../../../build/libvta.so  
INFO:RPCServer:load_module /tmp/tmpjpvsqhpe/conv2d.o  
INFO:root:Loading VTA library: /home/root/workspace/tvm/vta/python/vta/../../../build/libvta.so  
INFO:root:Loading VTA library: /home/root/workspace/tvm/vta/python/vta/../../../build/libvta.so  
INFO:root:Loading VTA library: /home/root/workspace/tvm/vta/python/vta/../../../build/libvta.so  
・・・省略・・・
INFO:root:Loading VTA library: /home/root/workspace/tvm/vta/python/vta/../../../build/libvta.so  
Process Process-1:2:  
Traceback (most recent call last):  
File "/usr/local/lib/python3.8/multiprocessing/process.py", line 315, in _bootstrap  
self.run()  
File "/usr/local/lib/python3.8/multiprocessing/process.py", line 108, in run  
self._target(\*self._args, \*\*self._kwargs)  
File "/home/root/workspace/tvm/python/tvm/rpc/server.py", line 118, in _serve_loop  
_ffi_api.ServerLoop(sockfd)  
File "/home/root/workspace/tvm/python/tvm/_ffi/_ctypes/packed_func.py", line 237, in _call_  
raise get_last_ffi_error()  
AttributeError: Traceback (most recent call last):  
4: TVMFuncCall  
3: std::_Function_handler<void (tvm::runtime::TVMArgs, tvm::runtime::TVMRetValue\*), tvm::runtime::{lambda(tvm::runtime::TVMArgs, tvm::runtime::TVMRetValue\*)#2}>::_M_invoke(std::_Any_data const&, tvm::runtime::TVMArgs&&, tvm::runtime::TVMRetValue\*&&)  
2: tvm::runtime::RPCServerLoop(int)  
1: tvm::runtime::RPCEndpoint::ServerLoop()  
0: std::_Function_handler<void (tvm::runtime::TVMArgs, tvm::runtime::TVMRetValue\*), TVMFuncCreateFromCFunc::{lambda(tvm::runtime::TVMArgs, tvm::runtime::TVMRetValue\*)#2}>::_M_invoke(std::_Any_data const&, tvm::runtime::TVMArgs&&, tvm::runtime::TVMRetValue\*&&)  
File "/home/root/workspace/tvm/python/tvm/_ffi/_ctypes/packed_func.py", line 81, in cfun  
rv = local_pyfunc(\*pyargs)  
File "/home/root/workspace/tvm/vta/python/vta/exec/rpc_server.py", line 84, in server_shutdown  
runtime_dll\[0\].VTARuntimeShutdown()  
File "/usr/local/lib/python3.8/ctypes/_init_.py", line 386, in _getattr_  
func = self._getitem_(name)  
File "/usr/local/lib/python3.8/ctypes/_init_.py", line 391, in _getitem_  
func = self._FuncPtr((name_or_ordinal, self))  
AttributeError: /home/root/workspace/tvm/vta/python/vta/../../../build/libvta.so: undefined symbol: VTARuntimeShutdown
```
