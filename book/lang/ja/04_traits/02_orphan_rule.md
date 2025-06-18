# トレイトの実装

型が別のクレート（例：Rustの標準ライブラリの`u32`）で定義されている場合、
その型に対して直接新しいメソッドを定義することはできません。もし試すと：

```rust
impl u32 {
    fn is_even(&self) -> bool {
        self % 2 == 0
    }
}
```

コンパイラは文句を言うでしょう：

```text
error[E0390]: cannot define inherent `impl` for primitive types
  |
1 | impl u32 {
  | ^^^^^^^^
  |
  = help: consider using an extension trait instead
```

## エクステンショントレイト

**エクステンショントレイト**は、`u32`のような外部の型に新しいメソッドを付与することを
主な目的とするトレイトです。
これは前の練習で使ったパターンとまさに同じで、`IsEven`トレイトを定義し、
それを`i32`と`u32`に実装しました。`IsEven`がスコープ内にある限り、
これらの型に対して`is_even`を自由に呼び出すことができます。

```rust
// トレイトをスコープに持ち込む
use my_library::IsEven;

fn main() {
    // それを実装している型でそのメソッドを呼び出す
    if 4.is_even() {
        // [...]
    }
}
```

## 一つの実装

書けるトレイト実装には制限があります。\
最もシンプルで直感的なもの：同じトレイトを同じ型に対して、一つのクレート内で
二度実装することはできません。

例えば：

```rust
trait IsEven {
    fn is_even(&self) -> bool;
}

impl IsEven for u32 {
    fn is_even(&self) -> bool {
        true
    }
}

impl IsEven for u32 {
    fn is_even(&self) -> bool {
        false
    }
}
```

コンパイラはこれを拒否します：

```text
error[E0119]: conflicting implementations of trait `IsEven` for type `u32`
   |
5  | impl IsEven for u32 {
   | ------------------- first implementation here
...
11 | impl IsEven for u32 {
   | ^^^^^^^^^^^^^^^^^^^ conflicting implementation for `u32`
```

`u32`値に対して`IsEven::is_even`が呼び出される際に、どのトレイト実装を使うべきかについて
曖昧さがあってはならないため、ただ一つしかありえません。

## オーファンルール

複数のクレートが関わる場合は、より複雑になります。
特に、以下のうち少なくとも一つは真でなければなりません：

- トレイトが現在のクレートで定義されている
- 実装者の型が現在のクレートで定義されている

これはRustの**オーファンルール**として知られています。その目的は、メソッド解決
プロセスを曖昧さのないものにすることです。

次のような状況を想像してみてください：

- クレート`A`が`IsEven`トレイトを定義
- クレート`B`が`u32`に対して`IsEven`を実装
- クレート`C`が`u32`に対して`IsEven`トレイトの（異なる）実装を提供
- クレート`D`が`B`と`C`の両方に依存し、`1.is_even()`を呼び出す

どの実装を使うべきでしょうか？`B`で定義されたもの？それとも`C`で定義されたもの？\
良い答えはありません。そのため、このシナリオを防ぐためにオーファンルールが定義されました。
オーファンルールのおかげで、クレート`B`もクレート`C`もコンパイルできません。

## さらなる読み物

- 上記で述べたオーファンルールには、いくつかの注意点と例外があります。
  その細部に慣れたい場合は、[リファレンス](https://doc.rust-lang.org/reference/items/implementations.html#trait-implementation-coherence)をチェックしてください。