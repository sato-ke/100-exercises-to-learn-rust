# `Drop`トレイト

[デストラクタ](../03_ticket_v1/11_destructor.md)を紹介したとき、
`drop`関数が次のことを行うと述べました：

1. 型が占有するメモリを回収する（つまり`std::mem::size_of`バイト）
2. 値が管理している可能性のある追加のリソースをクリーンアップする（例：`String`のヒープバッファ）

ステップ2が`Drop`トレイトの出番です。

```rust
pub trait Drop {
    fn drop(&mut self);
}
```

`Drop`トレイトは、コンパイラが自動的に行うもの以外に、型に対する_追加の_
クリーンアップロジックを定義するメカニズムです。\
`drop`メソッドに入れたものは何でも、値がスコープを外れるときに実行されます。

## `Drop`と`Copy`

`Copy`トレイトについて話すとき、型がメモリ内で占有する`std::mem::size_of`バイトを超えて
追加のリソースを管理する場合、`Copy`を実装できないと言いました。

疑問に思うかもしれません：コンパイラは型が追加のリソースを管理するかどうかをどのように知るのでしょうか？
その通りです：`Drop`トレイト実装です！\
型に明示的な`Drop`実装がある場合、コンパイラは
その型に追加のリソースが付随していると仮定し、`Copy`の実装を許可しません。

```rust
// これはユニット構造体、つまりフィールドのない構造体です。
#[derive(Clone, Copy)]
struct MyType;

impl Drop for MyType {
    fn drop(&mut self) {
       // ここで何もする必要はありません。
       // 「空の」Drop実装があるだけで十分です
    }
}
```

コンパイラは次のエラーメッセージで文句を言うでしょう：

```text
error[E0184]: the trait `Copy` cannot be implemented for this type; 
              the type has a destructor
 --> src/lib.rs:2:17
  |
2 | #[derive(Clone, Copy)]
  |                 ^^^^ `Copy` not allowed on types with destructors
```