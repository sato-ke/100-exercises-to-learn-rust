# 可変インデックス

`Index`は読み取り専用アクセスを許可します。取得した値を変更することはできません。

## `IndexMut`

可変性を許可したい場合は、`IndexMut`トレイトを実装する必要があります。

```rust
// 若干簡略化
pub trait IndexMut<Idx>: Index<Idx>
{
    // 必須メソッド
    fn index_mut(&mut self, index: Idx) -> &mut Self::Output;
}
```

`IndexMut`は型が既に`Index`を実装している場合にのみ実装できます。
これは_追加の_機能を解除するからです。