# Google/JAX調査(インストール編)


JAXとは、Python+NumPyプログラムを微分可能かつ、XLAを使ってGPUもしくはTPU上で動作するコードにコンパイルしてくれる、Pythonライブラリ。Googleが開発しており、現状はリサーチプロジェクト。
この記事ではJAX調査に向けて、WSL上でソースコードからビルドする環境構築手順を紹介。

# 依存ライブラリ導入

ビルドに必要なPythonとライブラリを導入。

```
sudo apt install -y python3 python3-dev
pip install --user scipy
```

# ソースコード取得

GitHubからソースコードを取得。

```
git clone <https://github.com/google/jax>
```

# ビルド開始

Pythonスクリプトを実行し、ビルドを開始します。XLAをCUDA有りでビルドする場合、`--enable_cuda`オプションを有効にして`build.py`を実行。以下はCPUのみのビルド。

```
cd jax
python build/build.py
```

Ryzen5 1600、メモリ32GB、1TB SSDの環境で２時間ぐらいかかった。

# インストール

以下のコマンドでインストール。

```
pip install -e build
pip install -e .
```

# 動作確認

Pythonインタプリタを起動後、JAXをインポートし、行列積の演算を実行。

```
$ python
Python 3.6.7 (default, Oct 22 2018, 11:32:17)
[GCC 8.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import jax.numpy as np
>>> from jax import random
>>> key = random.PRNGKey(0)
/home/mshr/src/github.com/google/jax/jax/lib/xla_bridge.py:164: UserWarning: No GPU found, falling back to CPU.
  warnings.warn('No GPU found, falling back to CPU.')
>>> x = random.normal(key, (5000, 5000))
>>> print(np.dot(x, x.T) / 2)
[[ 2.5272668e+03  8.1589289e+00 -8.5327983e-01 ... -9.3034286e+00
  -3.1621683e+01 -1.7542191e+01]
 [ 8.1589289e+00  2.4567305e+03  2.2345556e+01 ...  5.2793068e+01
   6.4099159e+00 -9.1594181e+00]
 [-8.5327983e-01  2.2345556e+01  2.4511853e+03 ...  1.2905287e+01
   2.2311758e+01 -2.4411291e+01]
 ...
 [-9.3034382e+00  5.2793064e+01  1.2905293e+01 ...  2.5079480e+03
  -4.8833957e+00 -2.1701965e+01]
 [-3.1621683e+01  6.4099188e+00  2.2311762e+01 ... -4.8834038e+00
   2.4965664e+03  2.1068907e+01]
 [-1.7542183e+01 -9.1594162e+00 -2.4411289e+01 ... -2.1701965e+01
   2.1068907e+01  2.4813088e+03]]
```

以上でGoogle/JAXをソースコードからビルドし、インストール、動作確認ができた。
