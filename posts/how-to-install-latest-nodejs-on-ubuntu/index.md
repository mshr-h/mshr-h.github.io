# Ubuntuで最新版Node.jsをインストールする


`apt`などのパッケージマネージャでnode.jsをインストールすると、古いバージョンが導入されることがある。[n package](https://github.com/tj/n)を使うことで、簡単にnode.jsの最新版を導入できる。

# 手順

1. node.jsとnpmを導入

    ```
    $ sudo apt install -y nodejs npm
    ```

2. n packageを導入

    ```
    $ sudo npm install -g n
    ```

3. n packageを使い、nodeの最新版を導入

    ```
    $ sudo n stable
    ```

4. 最後に、`apt-get`で導入したnode.jsとnpmを削除

    ```
    $ sudo apt purge -y nodejs npm
    ```

5. 正しく導入されていることを確認

    ```
    $ node -v
    v12.16.1
    ```
