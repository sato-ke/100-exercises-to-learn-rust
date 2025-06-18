# `From`と`Into`

文字列の旅が始まった場所に戻りましょう：

```rust
let ticket = Ticket::new(
    "A title".into(), 
    "A description".into(), 
    "To-Do".into()
);
```

ここで`.into()`が何をしているかを解き明かし始めるのに十分な知識が身につきました。

## 問題

これは`new`メソッドのシグネチャです：

```rust
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

文字列リテラル（`"A title"`など）が`&str`型であることも見てきました。\
ここで型の不一致があります：`String`が期待されているのに、`&str`があります。
今回は魔法の型強制が助けに来てくれません。**変換を実行する**必要があります。

## `From`と`Into`

Rust標準ライブラリは、**失敗しない変換**のために2つのトレイトを定義しています：
`std::convert`モジュールの`From`と`Into`です。

```rust
pub trait From<T>: Sized {
    fn from(value: T) -> Self;
}

pub trait Into<T>: Sized {
    fn into(self) -> T;
}
```

これらのトレイト定義は、まだ見たことのないいくつかの概念を示しています：
**スーパートレイト**と**暗黙のトレイト境界**です。
まずそれらを解き明かしましょう。

### スーパートレイト / サブトレイト

`From: Sized`構文は、`From`が`Sized`の**サブトレイト**であることを意味します：
`From`を実装する任意の型は、`Sized`も実装していなければなりません。
代わりに、`Sized`は`From`の**スーパートレイト**だと言うこともできます。

### 暗黙のトレイト境界

ジェネリック型パラメータを持つ場合は常に、コンパイラは暗黙的に
それが`Sized`だと仮定します。

例えば：

```rust
pub struct Foo<T> {
    inner: T,
}
```

これは実際には次と等価です：

```rust
pub struct Foo<T: Sized> 
{
    inner: T,
}
```

`From<T>`の場合、トレイト定義は次と等価です：

```rust
pub trait From<T: Sized>: Sized {
    fn from(value: T) -> Self;
}
```

言い換えれば、`T`と`From<T>`を実装する型の_両方_が`Sized`でなければならず、
前者の境界は暗黙的であっても同様です。

### 負のトレイト境界

**負のトレイト境界**で暗黙の`Sized`境界をオプトアウトできます：

```rust
pub struct Foo<T: ?Sized> {
    //            ^^^^^^^
    //            これは負のトレイト境界です
    inner: T,
}
```

この構文は「`T`は`Sized`かもしれないし、そうでないかもしれない」と読み、
`T`をDST（例：`Foo<str>`）に束縛することを可能にします。ただし、これは特別なケースです：
負のトレイト境界は`Sized`専用で、他のトレイトでは使用できません。

## `&str`から`String`へ

[`std`のドキュメント](https://doc.rust-lang.org/std/convert/trait.From.html#implementors)で、
どの`std`型が`From`トレイトを実装しているかを確認できます。\
`String`が`From<&str> for String`を実装していることがわかります。したがって、次のように書けます：

```rust
let title = String::from("A title");
```

しかし、主に`.into()`を使ってきました。\
[`Into`の実装者](https://doc.rust-lang.org/std/convert/trait.Into.html#implementors)をチェックしても、
`Into<String> for &str`は見つからないでしょう。何が起こっているのでしょうか？

`From`と`Into`は**双対トレイト**です。\
特に、`Into`は**ブランケット実装**を使用して`From`を実装する任意の型に対して実装されます：

```rust
impl<T, U> Into<U> for T
where
    U: From<T>,
{
    fn into(self) -> U {
        U::from(self)
    }
}
```

型`U`が`From<T>`を実装していれば、`Into<U> for T`が自動的に実装されます。そのため、
`let title = "A title".into();`と書くことができます。

## `.into()`

`.into()`を見るたびに、型間の変換が行われていることがわかります。\
しかし、対象の型は何でしょうか？

ほとんどの場合、対象の型は以下のいずれかです：

- 関数/メソッドのシグネチャで指定される（上記の例の`Ticket::new`など）
- 型注釈付きの変数宣言で指定される（例：`let title: String = "A title".into();`）

`.into()`は、コンパイラが曖昧さなしにコンテキストから対象の型を推論できる限り、
そのまま動作します。