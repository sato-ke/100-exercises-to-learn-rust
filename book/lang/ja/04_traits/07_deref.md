# `Deref`トレイト

前の練習では、あまりやることがありませんでしたね。

次を変更するだけでした：

```rust
impl Ticket {
    pub fn title(&self) -> &String {
        &self.title
    }
}
```

から

```rust
impl Ticket {
    pub fn title(&self) -> &str {
        &self.title
    }
}
```

コードをコンパイルしてテストを通すために必要だったのはこれだけでした。
しかし、あなたの頭の中では何らかの警鐘が鳴っているはずです。

## うまくいくはずがないのに、うまくいく

事実を整理しましょう：

- `self.title`は`String`です
- `&self.title`は、従って、`&String`です
- （修正された）`title`メソッドの出力は`&str`です

コンパイラエラーを期待するでしょう？`Expected &String, found &str`とか似たようなものを。
代わりに、ただうまくいきます。**なぜ**でしょうか？

## `Deref`の救済

`Deref`トレイトは、[**deref coercion**](https://doc.rust-lang.org/std/ops/trait.Deref.html#deref-coercion)として知られる言語機能の背後にあるメカニズムです。\
このトレイトは標準ライブラリの`std::ops`モジュールで定義されています：

```rust
// 今のところ定義を少し簡略化しています。
// 完全な定義は後で見ます。
pub trait Deref {
    type Target;
    
    fn deref(&self) -> &Self::Target;
}
```

`type Target`は**関連型**です。\
これは、トレイトが実装される際に指定されなければならない具象型のプレースホルダーです。

## Deref coercion

型`T`に対して`Deref<Target = U>`を実装することで、コンパイラに`&T`と`&U`が
ある程度交換可能であることを伝えています。\
特に、次の動作が得られます：

- `T`への参照は暗黙的に`U`への参照に変換される（つまり`&T`が`&U`になる）
- `&self`を入力として取る`U`で定義されたすべてのメソッドを`&T`で呼び出すことができる

参照外し演算子`*`に関してもう一つありますが、まだ必要ありません（興味があれば`std`のドキュメントを参照）。

## `String`は`Deref`を実装する

`String`は`Target = str`で`Deref`を実装します：

```rust
impl Deref for String {
    type Target = str;
    
    fn deref(&self) -> &str {
        // [...]
    }
}
```

この実装とderef coercionのおかげで、`&String`は必要に応じて自動的に`&str`に変換されます。

## Deref coercionを乱用しない

Deref coercionは強力な機能ですが、混乱を招く可能性があります。\
型を自動的に変換することで、コードが読みにくく理解しにくくなる可能性があります。
同じ名前のメソッドが`T`と`U`の両方で定義されている場合、どちらが呼び出されるでしょうか？

このコースで後ほど、deref coercionの「最も安全な」使用例を調べます：スマートポインタです。