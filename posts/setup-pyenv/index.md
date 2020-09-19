# pyenv導入手順メモ


pyenvとpyenv-virtualenvを導入する手順メモ。

## 環境

- Ubuntu 20.04 LTS

## 手順

```bash
sudo apt install -y build-essential libffi-dev libssl-dev zlib1g-dev liblzma-dev libbz2-dev libreadline-dev libsqlite3-dev git
git clone https://github.com/pyenv/pyenv.git ~/.pyenv
git clone https://github.com/pyenv/pyenv-virtualenv.git ~/.pyenv/plugins/pyenv-virtualenv
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc
echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bashrc
source ~/.bashrc
```

## Pythonインストール

`--enable-shared`オプションを付けて共有ライブラリをビルドする。

```bash
env PYTHON_CONFIGURE_OPTS="--enable-shared" pyenv install 3.8.5
pyenv virtualenv 3.8.5 env
pyenv global env
```

