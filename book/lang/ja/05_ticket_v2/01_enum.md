# 列挙型

[前の章](../03_ticket_v1/02_validation.md)で書いたバリデーションロジックに基づくと、チケットには有効なステータスが数種類しかありません：`To-Do`、`InProgress`、`Done`です。  
これは、`Ticket`構造体の`status`フィールドや、`new`メソッドの`status`パラメータの型を見ても明らかではありません：

```rust
#[derive(Debug, PartialEq)]
pub struct Ticket {
    title: String,
    description: String,
    status: String,
}

impl Ticket {
    pub fn new(
        title: String, 
        description: String, 
        status: String
    ) -> Self {
        // [...]
    }
}
```

どちらの場合も、`status`フィールドを表現するために`String`を使用しています。  
`String`は非常に汎用的な型で、`status`フィールドが限られた値のセットしか持たないという情報を即座に伝えません。さらに悪いことに、`Ticket::new`の呼び出し元は、提供したステータスが有効かどうかを**実行時**にしか知ることができません。

**列挙型**を使えば、これをより良くできます。

## `enum`

列挙型は、**バリアント**と呼ばれる固定された値のセットを持つことができる型です。  
Rustでは、`enum`キーワードを使用して列挙型を定義します：

```rust
enum Status {
    ToDo,
    InProgress,
    Done,
}
```

`enum`は、`struct`と同様に、**新しいRust型**を定義します。