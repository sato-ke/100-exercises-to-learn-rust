# インデックス

`TicketStore::get`は指定された`TicketId`に対して`Option<&Ticket>`を返します。\
配列やベクタの要素にRustのインデックス構文を使用してアクセスする方法を以前に見ました：

```rust
let v = vec![0, 1, 2];
assert_eq!(v[0], 0);
```

`TicketStore`に同じ体験を提供するにはどうしたらよいでしょうか？\
正解です：トレイト、`Index`を実装する必要があります！

## `Index`

`Index`トレイトはRustの標準ライブラリで定義されています：

```rust
// 若干簡略化
pub trait Index<Idx>
{
    type Output;

    // 必須メソッド
    fn index(&self, index: Idx) -> &Self::Output;
}
```

これには以下があります：

- インデックス型を表すジェネリックパラメータ`Idx`
- インデックスを使用して取得した型を表す関連型`Output`

`index`メソッドが`Option`を返さないことに注意してください。前提は、
配列やベクタのインデックスで起こるように、そこにない要素にアクセスしようとすると
`index`がパニックするということです。