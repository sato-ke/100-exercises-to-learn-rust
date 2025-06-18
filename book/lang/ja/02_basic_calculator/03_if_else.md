# 制御フロー、パート1

これまでのプログラムは非常に単純なものでした。\
命令のシーケンスが上から下へ実行され、それだけです。

**分岐**を導入する時が来ました。

## `if`句

`if`キーワードは、条件が真の場合にのみコードブロックを実行するために使用されます。

簡単な例を示します：

```rust
let number = 3;
if number < 5 {
    println!("`number` is smaller than 5");
}
```

このプログラムは、条件`number < 5`が真であるため、`number is smaller than 5`を出力します。

### `else`句

ほとんどのプログラミング言語と同様に、Rustは`if`式の条件が偽の場合にコードブロックを実行するオプションの`else`分岐をサポートしています。\
例えば：

```rust
let number = 3;

if number < 5 {
    println!("`number` is smaller than 5");
} else {
    println!("`number` is greater than or equal to 5");
}
```

### `else if`句

複数の`if`式を入れ子にすると、コードはどんどん右に寄っていきます。

```rust
let number = 3;

if number < 5 {
    println!("`number` is smaller than 5");
} else {
    if number >= 3 {
        println!("`number` is greater than or equal to 3, but smaller than 5");
    } else {
        println!("`number` is smaller than 3");
    }
}
```

`else if`キーワードを使用して、複数の`if`式を1つに組み合わせることができます：

```rust
let number = 3;

if number < 5 {
    println!("`number` is smaller than 5");
} else if number >= 3 {
    println!("`number` is greater than or equal to 3, but smaller than 5");
} else {
    println!("`number` is smaller than 3");
}
```

## ブール値

`if`式の条件は`bool`型、つまり**ブール値**でなければなりません。\
ブール値は、整数と同様に、Rustのプリミティブ型です。

ブール値は`true`または`false`の2つの値のいずれかを持つことができます。

### 真偽値への暗黙の変換はない

`if`式の条件がブール値でない場合、コンパイルエラーが発生します。

例えば、次のコードはコンパイルされません：

```rust
let number = 3;
if number {
    println!("`number` is not zero");
}
```

次のコンパイルエラーが発生します：

```text
error[E0308]: mismatched types
 --> src/main.rs:3:8
  |
3 |     if number {
  |        ^^^^^^ expected `bool`, found integer
```

これはRustの型強制に関する哲学に従っています：非ブール型からブール型への自動変換はありません。
RustにはJavaScriptやPythonのような**truthy**や**falsy**値の概念がありません。\
チェックしたい条件について明示的でなければなりません。

### 比較演算子

`if`式の条件を構築するために比較演算子を使用することは非常に一般的です。\
整数を扱う際にRustで使用できる比較演算子は次のとおりです：

- `==`: 等しい
- `!=`: 等しくない
- `<`: より小さい
- `>`: より大きい
- `<=`: 以下
- `>=`: 以上

## `if/else`は式

Rustでは、`if`式は**式**であり、文ではありません：値を返します。\
その値は変数に代入したり、他の式で使用したりできます。例えば：

```rust
let number = 3;
let message = if number < 5 {
    "smaller than 5"
} else {
    "greater than or equal to 5"
};
```

上記の例では、`if`の各分岐が文字列リテラルに評価され、
それが`message`変数に代入されます。\
唯一の要件は、両方の`if`分岐が同じ型を返すことです。