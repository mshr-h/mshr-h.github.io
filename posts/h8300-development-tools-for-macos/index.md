# H8/3069Fの開発ツールをmacOSに導入


H8/3069Fの開発ツールを導入する。次のソースコードをダウンロードし、`$HOME/h8devtools`に保存する。

- [GMP](https://gmplib.org/)
- [MPFR](https://gforge.inria.fr/frs/?group_id=136&release_id=10670#mpfr-_4.0.2-title-content)
- [binutils](ftp://sourceware.org/pub/binutils/snapshots)
- [MPC](http://repository.timesys.com/buildsources/m/mpc/)
- [GCC](http://ftp.gnu.org/gnu/gcc/)
- [h8write](http://mes.osdn.jp/h8/writer-j.html)

あとはひたすら`configure`&`make`する。

```bash
cd ~/h8devtools/
tar xvf gmp-6.2.0.tar.lz
cd gmp-6.2.0
mkdir build && cd build
../configure --prefix=$HOME/h8devtools
make -j4
make install
cd ../..

tar xvf mpfr-4.0.2.tar.bz2
cd mpfr-4.0.2
mkdir build && cd build
../configure --prefix=$HOME/h8devtools --with-gmp=$HOME/h8devtools
make -j4
make install
cd ../..

tar xvf binutils-2.33.90.tar.xz
cd binutils-2.33.90
mkdir build && cd build
../configure --prefix=$HOME/h8devtools --target=h8300-elf --disable-nls
make -j4
make install
cd ../..

tar xvf mpc-1.1.0.tar.gz
cd mpc-1.1.0
mkdir build && cd build
../configure --prefix=$HOME/h8devtools --with-gmp=$HOME/h8devtools --with-mpfr=$HOME/h8devtools
make -j4
make install
cd ../..

tar xvf gcc-9.3.0.tar.xz
cd gcc-9.3.0
mkdir build && cd build
../configure --target=h8300-elf --disable-nls --disable-threads --disable-shared --disable-libssp --enable-languages=c --with-gmp=$HOME/h8devtools --with-mpfr=$HOME/h8devtools --with-mpc=$HOME/h8devtools --prefix=$HOME/h8devtools
make -j4
make install

cd ../..
clang -Wall -o h8write h8write.c
mv h8write $HOME/h8devtools/bin
```

最後に、`$HOME/h8devtools/bin`を環境変数PATHへ追加する。`h8300-elf-gcc`が実行できれば正常に開発ツールが導入されている。

```bash
mshrs-MacBook-Pro:~ mshr$ h8300-elf-gcc -v
Using built-in specs.
COLLECT_GCC=h8300-elf-gcc
COLLECT_LTO_WRAPPER=/Users/mshr/h8devtools/libexec/gcc/h8300-elf/9.3.0/lto-wrapper
Target: h8300-elf
Configured with: ../configure --target=h8300-elf --disable-nls --disable-threads --disable-shared --disable-libssp --enable-languages=c --with-gmp=/Users/mshr/h8devtools --with-mpfr=/Users/mshr/h8devtools --with-mpc=/Users/mshr/h8devtools --prefix=/Users/mshr/h8devtools
Thread model: single
gcc version 9.3.0 (GCC)
```
