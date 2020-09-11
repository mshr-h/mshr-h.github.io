# Rustの所有権、借用、ライフタイムについて雑に理解した


最近、Rustプログラミング言語に入門した。Rustの公式ページには、Rustの特徴として信頼性について記載されている。本記事では、Rustが保証するメモリ安全性とスレッド安全性を実現する、所有権システムについて雑に理解したので説明する。

# 所有権システム

Rustは、コンパイル時にメモリ安全性に関するバグをコンパイルエラーとして開発者に教えてくれる。所有権システムは、メモリ安全性に関するバグをRustコンパイラが発見するための仕組み。所有権システムは、次の3つの概念で構成されている。

- 所有権
- 借用
- ライフタイム

# 所有権

変数束縛において、Rustでは所有権という特性を持つ。変数は、束縛している値の「所有権」を持つ。Rustの値は、所有権を持つ変数がスコープから外れると、その値のリソースは開放される。

例えば以下の例では、内側の`{}`に入ると、新しい値`Vec<T>`が作成され、変数`bar`が値を束縛する。変数`bar`のスコープは内側の`{}`の間のため、`{}`の前でも後でも使えない。内側の`{}`を抜けると、変数`bar`のスコープが終了し、値`Vec<T>`のリソースが開放(デストラクタ`drop()`が呼ばれる)される。

```rust
fn foo() {
    // barは使えない
    {
        let bar = vec![1, 2, 3];
        println!("bar.0 = {}", bar.0);
    }
    // Vec<T>のリソースが開放される
    // barは使えない
}
```

## ムーブとコピー

次のコードを実行すると期待通りの実行結果が得られる。

```rust
fn main() {
    let x = 1;
    let y = x;
    println!("x = {}, y = {}", x, y);
    // => x = 1, y = 1
}
```

一方、次のコードではコンパイルエラーが発生する。

```rust
fn main() {
    let x = String::from("hello");
    let y = x;
    println!("x = {}, y = {}", x, y);
}
```

```rust
error[E0382]: borrow of moved value: `x`
 --> src/main.rs:4:32
  |
2 |     let x = String::from("hello");
  |         - move occurs because `x` has type `std::string::String`, which does not implement the `Copy` trait
3 |     let y = x;
  |             - value moved here
4 |     println!("x = {}, y = {}", x, y);
  |                                ^ value borrowed here after move
```

Rustでは、関数の引数や代入文は値そのものを渡すのではなく、所有権をムーブするのがデフォルト。`let y = x`で`std::string::String`の所有権は`y`にムーブされたにも関わらず、`println!("x = {}, y = {}", x, y)`で`x`を使おうとしたため、エラーが発生した。

Rustのプリミティブ型のうち、スカラ型にはCopyトレイトが実装されている。Copyトレイトが実装されている変数の場合、値をコピーして、新しくその値の所有権を持つ変数を作成する。最初の例では、`x`はCopyトレイトが実装されているため、`let y = x`は所有権のムーブではなく、コピーされ、後続の処理においても`x`を使うことができる。

## 関数をまたいだ所有権の移動

関数呼び出しにおける引数においても、所有権はムーブする。次の例では、`print_string(str)`で`str`を引数として指定しているため、所有権もムーブする。そのため、`print_string(str)`以降の処理では`str`を使用することができない。

```rust
fn print_string(x: String) {
    println!("{}", x);
}

fn main() {
    let str = String::from("sample");
    print_string(str);
    // strの所有権はprint_stinr()に移動したため、以降strは使えない
}
```

次のように、`print_string()`で`x`の所有権を返し、`main()`で所有権を受け取ることで解決できる。しかし、このような書き方は煩雑のため、「借用」を使うことで同様のことを実現する仕組みがある。

```rust
fn print_string(x: String) -> String{
    println!("{}", x);
    x
}

fn main() {
    let str = String::from("sample");
    let str2 = print_string(str);
    println!("{}", str2);
    // => sampleと出力される
}
```

# 借用

借用とは、所有権をムーブせずに、所有権を一時的に貸し出す参照を作る仕組み。参照は、スコープが終了してもリソースを開放しない。上記の例を借用を用いて書き直したのが以下のプログラム。

```rust
fn print_string(x: &String) {
    println!("{}", x);
}

fn main() {
    let str = String::from("sample");
    print_string(&str);
    println!("{}", str);
}
```

`&`は参照を意味する。参照は何らかの実態を指すポインタで、デフォルトでイミュータブルのため、参照している値を変更できない。変更するには、`&mut`を付けてミュータブルな参照を作る必要がある。

```rust
fn print_string(x: &mut String) {
    x.push_str(" text");
}

fn main() {
    let mut str = String::from("sample");
    println!("{}", str);
    print_string(&mut str);
    println!("{}", str);
}
```

借用は、次のいずれか一方の状態を満たす必要がある。

- ただ一つのミュータブルな参照が存在する状態

    ```rust
    fn main() {
        let mut str = String::from("sample");
        let str1 = &mut str;
        str1.push_str(" text");
        println!("str1: {}", str1);
        // => sample text
    }
    ```

- 複数のイミュータブルな参照が存在する状態

    ```rust
    fn main() {
        let mut str = String::from("sample");
        let str1 = &str;
        let str2 = &str;
        println!("str1: {}, str2: {}", str1, str2);
        // => str1: sample, str2: sample
    }
    ```

# ライフタイム

ライフタイムとは、参照が有効になる範囲(スコープ)のこと。参照は、何らかの実体を指すポインタのため、その指している実体より長く生き残ることはできない。ライフタイムにより、ある参照には必ず実体が存在する、ということが保証される。多くの場合、プログラマはRustコンパイラがライフタイムを推論してくれるため、ライフタイムを明示的に指定する必要はない。Rustコンパイラは、借用チェッカにより参照のスコープを比較することで、参照が有効かをチェックする。

例えば、下記のプログラムはコンパイルエラーが発生する。

```rust
fn main() {
    let y_ref;
    {
        let y = 10;
        y_ref = &y;
    }
    println!("x = {}", y_ref);
}
```

```rust
error[E0597]: `y` does not live long enough
 --> src/main.rs:5:17
  |
5 |         y_ref = &y;
  |                 ^^ borrowed value does not live long enough
6 |     }
  |     - `y` dropped here while still borrowed
7 |     println!("x = {}", y_ref);
  |                        ----- borrow later used here
```

これは、`y`のライフタイムは内側の`{}`であるのに対し、`y`を参照する`y_ref`はその外側であり、`y`は`y_ref`より長く生き残れないため、借用チェッカによりエラーが発生する。

# まとめ

Rustの所有権システムは、所有権、借用、ライフタイムの3つの概念から構成されている。

「所有権」とは、ある値は一つの変数のみからしか読み書きを行わない、というルールのこと。所有権を持つ変数のスコープが終了すると、その値は破棄される。ある変数が所有する所有権は、他の変数へ移動(ムーブ)することができ、ムーブすると元の変数から値にアクセスできなくなる。

ムーブすることなく、値へアクセスするために、「借用」という所有権を一時的に貸し出す仕組みにより、所有権を持つ変数への参照を作ることができる。借用は、「ただ一つのミュータブルな参照が存在する状態」、「複数のイミュータブルな参照が存在する状態」のいずれか一つの状態を満たすことができる。

参照には「ライフタイム」が存在する。ライフタイムとは、その参照が有効であるスコープのこと。Rustコンパイラは、ライフタイムを比較することで、参照が有効であることを保証する。
