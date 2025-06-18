# 順序

`Vec`から`HashMap`に移行することで、チケット管理システムのパフォーマンスを向上させ、
過程でコードを簡略化しました。\
しかし、すべてがバラ色ではありません。`Vec`に支えられたストアを反復する際は、チケットが
追加された順序で返されることを確信できました。
`HashMap`の場合はそうではありません：チケットを反復できますが、順序はランダムです。

`HashMap`から`BTreeMap`に切り替えることで、一貫した順序を取り戻すことができます。

## `BTreeMap`

`BTreeMap`は、エントリがキーによってソートされることを保証します。\
これは、エントリを特定の順序で反復する必要がある場合や、範囲クエリを実行する必要がある場合（例：「10から20の間のIDを持つすべてのチケットを取得」）に役立ちます。

`HashMap`と同様に、`BTreeMap`の定義にはトレイト境界は見つかりません。
しかし、そのメソッドにはトレイト境界があります。`insert`を見てみましょう：

```rust
// `K`と`V`はそれぞれキーと値の型を表します。
// `HashMap`と同様です。
impl<K, V> BTreeMap<K, V> {
    pub fn insert(&mut self, key: K, value: V) -> Option<V>
    where
        K: Ord,
    {
        // implementation
    }
}
```

`Hash`はもはや必要ありません。代わりに、キー型は`Ord`トレイトを実装しなければなりません。

## `Ord`

`Ord`トレイトは値を比較するために使用されます。\
`PartialEq`が等価性の比較に使用される一方で、`Ord`は順序の比較に使用されます。

これは`std::cmp`で定義されています：

```rust
pub trait Ord: Eq + PartialOrd {
    fn cmp(&self, other: &Self) -> Ordering;
}
```

`cmp`メソッドは`Ordering`列挙型を返します。これは
`Less`、`Equal`、または`Greater`のいずれかになります。\
`Ord`は他の2つのトレイトが実装されることを要求します：`Eq`と`PartialOrd`。

## `PartialOrd`

`PartialOrd`は、`PartialEq`が`Eq`の弱いバージョンであるのと同様に、`Ord`の弱いバージョンです。
その定義を見ることで理由がわかります：

```rust
pub trait PartialOrd: PartialEq {
    fn partial_cmp(&self, other: &Self) -> Option<Ordering>;
}
```

`PartialOrd::partial_cmp`は`Option`を返します—2つの値が
比較できることが保証されていません。\
例えば、`f32`は`NaN`値が比較できないため、`Ord`を実装しません。
これは`f32`が`Eq`を実装しない理由と同じです。

## `Ord`と`PartialOrd`の実装

`Ord`と`PartialOrd`の両方を型に対して派生できます：

```rust
// `Ord`が必要とするため、`Eq`と`PartialEq`も追加する必要があります。
#[derive(Eq, PartialEq, Ord, PartialOrd)]
struct TicketId(u64);
```

手動で実装することを選択する（または必要がある）場合は、注意してください：

- `Ord`と`PartialOrd`は`Eq`と`PartialEq`と一貫している必要があります。
- `Ord`と`PartialOrd`は互いに一貫している必要があります。