# ループ、パート1：`while`

`factorial`の実装では再帰を使用せざるを得ませんでした。\
特に関数型プログラミングのバックグラウンドから来ている場合、これは自然に感じるかもしれません。
または、CやPythonのようなより命令型の言語に慣れている場合は、奇妙に感じるかもしれません。

代わりに**ループ**を使用して同じ機能を実装する方法を見てみましょう。

## `while`ループ

`while`ループは、**条件**が真である限りコードブロックを実行する方法です。\
一般的な構文は次のとおりです：

```rust
while <条件> {
    // 実行するコード
}
```

例えば、1から5までの数値を合計したい場合：

```rust
let sum = 0;
let i = 1;
// "iが5以下の間"
while i <= 5 {
    // `+=`は`sum = sum + i`の短縮形です
    sum += i;
    i += 1;
}
```

これは、`i`が5以下でなくなるまで、`i`に1を加え、`sum`に`i`を加え続けます。

## `mut`キーワード

上記の例はそのままではコンパイルされません。次のようなエラーが表示されます：

```text
error[E0384]: cannot assign twice to immutable variable `sum`
 --> src/main.rs:7:9
  |
2 |     let sum = 0;
  |         ---
  |         |
  |         first assignment to `sum`
  |         help: consider making this binding mutable: `mut sum`
...
7 |         sum += i;
  |         ^^^^^^^^ cannot assign twice to immutable variable

error[E0384]: cannot assign twice to immutable variable `i`
 --> src/main.rs:8:9
  |
3 |     let i = 1;
  |         -
  |         |
  |         first assignment to `i`
  |         help: consider making this binding mutable: `mut i`
...
8 |         i += 1;
  |         ^^^^^^ cannot assign twice to immutable variable
```

これは、Rustの変数がデフォルトで**不変**であるためです。\
一度値が代入されると、その値を変更することはできません。

変更を許可したい場合は、`mut`キーワードを使用して変数を**可変**として宣言する必要があります：

```rust
// `sum`と`i`は今可変です！
let mut sum = 0;
let mut i = 1;

while i <= 5 {
    sum += i;
    i += 1;
}
```

これでエラーなしでコンパイルして実行できます。

## さらに読む

- [`while`ループのドキュメント](https://doc.rust-lang.org/std/keyword.while.html)