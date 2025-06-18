# トレイト境界

これまでのところ、トレイトの2つの使用例を見てきました：

- 「組み込み」動作の解禁（例：演算子オーバーロード）
- 既存の型への新しい動作の追加（例：エクステンショントレイト）

3つ目の使用例があります：**ジェネリックプログラミング**です。

## 問題

これまでの関数とメソッドはすべて、**具象型**で動作してきました。\
具象型で動作するコードは、通常書くのも理解するのも簡単です。しかし、
再利用性の面では制限があります。\
例えば、整数が偶数かどうかを返すTrueを返す関数を書きたいとしましょう。
具象型で作業する場合、サポートしたい各整数型に対して別々の関数を書く必要があります：

```rust
fn is_even_i32(n: i32) -> bool {
    n % 2 == 0
}

fn is_even_i64(n: i64) -> bool {
    n % 2 == 0
}

// その他
```

または、単一のエクステンショントレイトを書いて、各整数型に対して異なる実装を書くこともできます：

```rust
trait IsEven {
    fn is_even(&self) -> bool;
}

impl IsEven for i32 {
    fn is_even(&self) -> bool {
        self % 2 == 0
    }
}

impl IsEven for i64 {
    fn is_even(&self) -> bool {
        self % 2 == 0
    }
}

// その他
```

重複は残ります。

## ジェネリックプログラミング

**ジェネリクス**を使ってより良いことができます。\
ジェネリクスを使うと、具象型の代わりに**型パラメータ**で動作するコードを書けます：

```rust
fn print_if_even<T>(n: T)
where
    T: IsEven + Debug
{
    if n.is_even() {
        println!("{n:?} is even");
    }
}
```

`print_if_even`は**ジェネリック関数**です。\
特定の入力型に縛られていません。代わりに、以下を満たす任意の型`T`で動作します：

- `IsEven`トレイトを実装している
- `Debug`トレイトを実装している

この契約は**トレイト境界**で表現されます：`T: IsEven + Debug`。\
`+`記号は、`T`が複数のトレイトを実装することを要求するために使われます。`T: IsEven + Debug`は
「`T`が`IsEven`**かつ**`Debug`を実装している」と等価です。

## トレイト境界

`print_if_even`でのトレイト境界はどんな目的を果たしているのでしょうか？\
調べるために、それらを削除してみましょう：

```rust
fn print_if_even<T>(n: T) {
    if n.is_even() {
        println!("{n:?} is even");
    }
}
```

このコードはコンパイルされません：

```text
error[E0599]: no method named `is_even` found for type parameter `T` 
              in the current scope
 --> src/lib.rs:2:10
  |
1 | fn print_if_even<T>(n: T) {
  |                  - method `is_even` not found 
  |                    for this type parameter
2 |     if n.is_even() {
  |          ^^^^^^^ method not found in `T`

error[E0277]: `T` doesn't implement `Debug`
 --> src/lib.rs:3:19
  |
3 |         println!("{n:?} is even");
  |                   ^^^^^ 
  |   `T` cannot be formatted using `{:?}` because 
  |         it doesn't implement `Debug`
  |
help: consider restricting type parameter `T`
  |
1 | fn print_if_even<T: std::fmt::Debug>(n: T) {
  |                   +++++++++++++++++
```

トレイト境界がなければ、コンパイラは`T`が何を**できるか**わかりません。\
`T`に`is_even`メソッドがあることも、`T`を印刷用にフォーマットする方法もわかりません。
コンパイラの観点からは、裸の`T`は何の動作も持ちません。\
トレイト境界は、関数本体で必要な動作が存在することを保証することで、
使用できる型のセットを制限します。

## 構文：トレイト境界のインライン化

上記のすべての例では、トレイト境界を指定するために**`where`句**を使いました：

```rust
fn print_if_even<T>(n: T)
where
    T: IsEven + Debug
//  ^^^^^^^^^^^^^^^^^
//  これは`where`句です
{
    // [...]
}
```

トレイト境界がシンプルな場合、型パラメータの隣に直接**インライン**できます：

```rust
fn print_if_even<T: IsEven + Debug>(n: T) {
    //           ^^^^^^^^^^^^^^^^^
    //           これはインライントレイト境界です
    // [...]
}
```

## 構文：意味のある名前

上記の例では、型パラメータ名として`T`を使いました。これは関数が
型パラメータを1つだけ持つ場合の共通の慣習です。\
ただし、より意味のある名前を使うことを妨げるものはありません：

```rust
fn print_if_even<Number: IsEven + Debug>(n: Number) {
    // [...]
}
```

複数の型パラメータが関わる場合や、`T`という名前が型の役割について
十分な情報を伝えない場合には、意味のある名前を使うことが実際に**望ましい**です。
変数や関数パラメータに対してと同じように、型パラメータに名前を付ける際には
明確性と可読性を最大化してください。
ただし、Rustの慣習に従ってください：[型パラメータ名にはアッパーキャメルケースを使う](https://rust-lang.github.io/api-guidelines/naming.html#casing-conforms-to-rfc-430-c-case)。

## 関数のシグネチャが王様

なぜトレイト境界が必要なのか疑問に思うかもしれません。コンパイラは関数の本体から必要なトレイトを推論できないのでしょうか？\
できますが、しません。\
理由は[関数パラメータの明示的な型注釈](../02_basic_calculator/02_variables.md#function-arguments-are-variables)と同じです：
各関数のシグネチャは呼び出し側と被呼び出し側の間の契約であり、条件は明示的に述べられなければなりません。
これにより、より良いエラーメッセージ、より良いドキュメント、バージョン間での意図しない破壊の減少、
そしてより高速なコンパイル時間が可能になります。