# トレイト

`Ticket`型をもう一度見てみましょう：

```rust
pub struct Ticket {
    title: String,
    description: String,
    status: String,
}
```

これまでのテストはすべて、`Ticket`のフィールドを使ってアサーションを行ってきました。

```rust
assert_eq!(ticket.title(), "A new title");
```

2つの`Ticket`インスタンスを直接比較したい場合はどうでしょうか？

```rust
let ticket1 = Ticket::new(/* ... */);
let ticket2 = Ticket::new(/* ... */);
ticket1 == ticket2
```

コンパイラは私たちを止めるでしょう：

```text
error[E0369]: binary operation `==` cannot be applied to type `Ticket`
  --> src/main.rs:18:13
   |
18 |     ticket1 == ticket2
   |     ------- ^^ ------- Ticket
   |     |
   |     Ticket
   |
note: an implementation of `PartialEq` might be missing for `Ticket`
```

`Ticket`は新しい型です。何もしなければ、**それに付随する動作はありません**。\
Rustは、`String`を含んでいるというだけで、2つの`Ticket`インスタンスをどう比較すべきかを魔法のように推測したりしません。

ただし、Rustコンパイラは正しい方向に向かうよう促してくれています：`PartialEq`の実装が欠けているかもしれないと示唆しています。`PartialEq`は**トレイト**です！

## トレイトとは何か？

トレイトは、Rustにおける**インターフェース**を定義する方法です。\
トレイトは、ある型がそのトレイトの契約を満たすために実装しなければならないメソッドのセットを定義します。

### トレイトの定義

トレイト定義の構文は次のようになります：

```rust
trait <TraitName> {
    fn <method_name>(<parameters>) -> <return_type>;
}
```

例えば、実装者に`is_zero`メソッドの定義を要求する`MaybeZero`という名前のトレイトを定義することができます：

```rust
trait MaybeZero {
    fn is_zero(self) -> bool;
}
```

### トレイトの実装

型に対してトレイトを実装するには、通常の[^inherent]メソッドと同じように`impl`キーワードを使いますが、
構文は少し異なります：

```rust
impl <TraitName> for <TypeName> {
    fn <method_name>(<parameters>) -> <return_type> {
        // メソッドの本体
    }
}
```

例えば、カスタム数値型`WrappingU32`に対して`MaybeZero`トレイトを実装するには：

```rust
pub struct WrappingU32 {
    inner: u32,
}

impl MaybeZero for WrappingU32 {
    fn is_zero(self) -> bool {
        self.inner == 0
    }
}
```

### トレイトメソッドの呼び出し

トレイトメソッドを呼び出すには、通常のメソッドと同じように`.`演算子を使います：

```rust
let x = WrappingU32 { inner: 5 };
assert!(!x.is_zero());
```

トレイトメソッドを呼び出すには、2つの条件が真でなければなりません：

- 型がそのトレイトを実装していること
- トレイトがスコープ内にあること

後者を満たすために、トレイトに対して`use`文を追加する必要があるかもしれません：

```rust
use crate::MaybeZero;
```

これは以下の場合には必要ありません：

- トレイトが呼び出しが発生するのと同じモジュールで定義されている場合
- トレイトが標準ライブラリの**プレリュード**で定義されている場合
  プレリュードは、すべてのRustプログラムに自動的にインポートされるトレイトと型のセットです。
  すべてのRustモジュールの最初に`use std::prelude::*;`が追加されているかのような動作をします。

プレリュードに含まれるトレイトと型のリストは、
[Rustドキュメント](https://doc.rust-lang.org/std/prelude/index.html)で確認できます。

[^inherent]: トレイトを使わずに型に直接定義されたメソッドは、**固有メソッド**とも呼ばれます。