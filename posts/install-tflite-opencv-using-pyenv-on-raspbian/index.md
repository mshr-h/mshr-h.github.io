# RaspbianにTensorFlow Lite 2.1とOpenCV 4を導入する


[前回の記事]({{< ref "/posts/2020-04-25-install-raspbian-on-raspberrypi-3.md" >}})で導入したRaspbian Busterに、TensorFlow LiteとOpenCVの環境を構築したのでメモ。

# Python 3.7.7導入

```bash
$ sudo apt-get install -y git openssl libssl-dev libbz2-dev libreadline-dev libsqlite3-dev
$ sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev \
    libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev \
    xz-utils tk-dev libffi-dev liblzma-dev python-openssl
$ git clone https://github.com/pyenv/pyenv.git ~/.pyenv
$ git clone https://github.com/yyuu/pyenv-virtualenv.git ~/.pyenv/plugins/pyenv-virtualenv
$ sudo vi ~/.bashrc

# 以下を追記
# ここから
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
# ここまで

$ source ~/.bashrc
```

# 開発用Python仮想環境構築

開発用ディレクトリは`~/mldev`とする。

```bash
$ pyenv install 3.7.7
$ pyenv virtualenv 3.7.7 mldev
$ mkdir ~/mldev
$ cd ~/mldev
$ pyenv local mldev
```

# numpy導入

1.18.3は導入できなかったので、1つ古い1.18.2を導入する。

```bash
$ pip install https://www.piwheels.org/simple/numpy/numpy-1.18.2-cp37-cp37m-linux_armv7l.whl
```

# TensorFlow Lite 2.1導入

```bash
$ cd ~/mldev
$ pip install https://dl.google.com/coral/python/tflite_runtime-2.1.0.post1-cp37-cp37m-linux_armv7l.whl
```

# OpenCV 4導入

```bash
$ sudo apt-get install -y \
    build-essential cmake pkg-config \
    libjpeg-dev libtiff5-dev libpng-dev \
    libavcodec-dev libavformat-dev libswscale-dev libv4l-dev \
    libxvidcore-dev libx264-dev \
    libgtk2.0-dev libgtk-3-dev \
    libcanberra-gtk* \
    gfortran liblapacke-dev \
    libavresample-dev libtesseract-dev libleptonica-dev \
    libgstreamer1.0-dev  libgstreamer-plugins-base1.0-dev \
    libhdf5-dev libhdf5-serial-dev libhdf5-103 \
    libqtgui4 libqtwebkit4 libqt4-test \
    python3-pyqt5 libatlas-base-dev libjasper-dev
$ pip install opencv-contrib-python opencv-python
```

`.bashrc`に以下を追記し、端末を再起動する。これを追記しないと、OpenCVのインポート時に`opencv: undefined symbol: __atomic_fetch_add_8`のエラーがでる。

```bash
export LD_PRELOAD=/usr/lib/arm-linux-gnueabihf/libatomic.so.1
```

# 実行確認

```bash
$ cd ~/mldev
$ python
Python 3.7.7 (default, Apr 23 2020, 21:26:28)
[GCC 8.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import tflite_runtime
>>> import numpy
>>> import cv2
>>>
```
