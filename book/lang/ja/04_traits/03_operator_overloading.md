# 演算子オーバーロード

トレイトが何であるかの基本的な理解を得たところで、**演算子オーバーロード**に戻りましょう。
演算子オーバーロードは、`+`、`-`、`*`、`/`、`==`、`!=`などの演算子にカスタム動作を定義する能力です。

## 演算子はトレイト

Rustでは、演算子はトレイトです。\
各演算子に対して、その演算子の動作を定義する対応するトレイトがあります。
その型にそのトレイトを実装することで、対応する演算子の使用を**解禁**します。

例えば、[`PartialEq`トレイト](https://doc.rust-lang.org/std/cmp/trait.PartialEq.html)は
`==`と`!=`演算子の動作を定義します：

```rust
// Rustの標準ライブラリからの`PartialEq`トレイト定義
// （今のところ*少し*簡略化されています）
pub trait PartialEq {
    // 必要なメソッド
    //
    // `Self`は「そのトレイトを実装している型」を
    // 表すRustのキーワードです
    fn eq(&self, other: &Self) -> bool;

    // 提供されるメソッド
    fn ne(&self, other: &Self) -> bool { ... }
}
```

`x == y`と書くと、コンパイラは`x`と`y`の型に対する`PartialEq`トレイトの実装を探し、
`x == y`を`x.eq(y)`に置き換えます。これは構文糖衣です！

主要な演算子に対する対応は次のとおりです：

| 演算子                   | トレイト                                                                |
| ------------------------ | ----------------------------------------------------------------------- |
| `+`                      | [`Add`](https://doc.rust-lang.org/std/ops/trait.Add.html)               |
| `-`                      | [`Sub`](https://doc.rust-lang.org/std/ops/trait.Sub.html)               |
| `*`                      | [`Mul`](https://doc.rust-lang.org/std/ops/trait.Mul.html)               |
| `/`                      | [`Div`](https://doc.rust-lang.org/std/ops/trait.Div.html)               |
| `%`                      | [`Rem`](https://doc.rust-lang.org/std/ops/trait.Rem.html)               |
| `==`と`!=`               | [`PartialEq`](https://doc.rust-lang.org/std/cmp/trait.PartialEq.html)   |
| `<`、`>`、`<=`、`>=`     | [`PartialOrd`](https://doc.rust-lang.org/std/cmp/trait.PartialOrd.html) |

算術演算子は[`std::ops`](https://doc.rust-lang.org/std/ops/index.html)モジュールに、
比較演算子は[`std::cmp`](https://doc.rust-lang.org/std/cmp/index.html)モジュールにあります。

## デフォルト実装

`PartialEq::ne`のコメントは、「`ne`は提供されるメソッド」だと述べています。\
これは、`PartialEq`がトレイト定義の中で`ne`に対する**デフォルト実装**を提供していることを意味します。
定義スニペットでは`{ ... }`で省略されたブロックです。\
省略されたブロックを展開すると、次のようになります：

```rust
pub trait PartialEq {
    fn eq(&self, other: &Self) -> bool;

    fn ne(&self, other: &Self) -> bool {
        !self.eq(other)
    }
}
```

期待通りです：`ne`は`eq`の否定です。\
デフォルト実装が提供されているので、あなたの型に`PartialEq`を実装する際に`ne`の実装をスキップできます。
`eq`を実装するだけで十分です：

```rust
struct WrappingU8 {
    inner: u8,
}

impl PartialEq for WrappingU8 {
    fn eq(&self, other: &WrappingU8) -> bool {
        self.inner == other.inner
    }
    
    // ここに`ne`の実装はありません
}
```

ただし、デフォルト実装を使うことを強制されるわけではありません。
トレイトを実装する際にそれを上書きすることを選択できます：

```rust
struct MyType;

impl PartialEq for MyType {
    fn eq(&self, other: &MyType) -> bool {
        // カスタム実装
    }

    fn ne(&self, other: &MyType) -> bool {
        // カスタム実装
    }
}
```