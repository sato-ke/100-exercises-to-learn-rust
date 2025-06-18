# 値のコピー、パート1

前の章でオーナーシップと借用を紹介しました。\
特に、次のことを述べました：

- Rustのすべての値は、任意の時点で単一の所有者を持つ
- 関数が値の所有権を取る場合（「消費する」）、呼び出し元はその値をもう使用できない

これらの制限は多少制限的です。\
時には値の所有権を取る関数を呼び出さなければならないが、その後でも
その値を使用する必要がある場合があります。

```rust
fn consumer(s: String) { /* */ }

fn example() {
     let mut s = String::from("hello");
     consumer(s);
     s.push_str(", world!"); // error: value borrowed here after move
}
```

そこで`Clone`の出番です。

## `Clone`

`Clone`はRustの標準ライブラリで定義されているトレイトです：

```rust
pub trait Clone {
    fn clone(&self) -> Self;
}
```

そのメソッド`clone`は`self`への参照を取り、同じ型の新しい**所有された**インスタンスを返します。

## 実際に

上記の例に戻って、`consumer`を呼び出す前に新しい`String`インスタンスを作成するために`clone`を使用できます：

```rust
fn consumer(s: String) { /* */ }

fn example() {
     let mut s = String::from("hello");
     let t = s.clone();
     consumer(t);
     s.push_str(", world!"); // エラーなし
}
```

`s`の所有権を`consumer`に渡す代わりに、新しい`String`を（`s`をクローンすることで）作成し、
それを`consumer`に渡します。\
`s`は`consumer`の呼び出し後も有効で使用可能です。

## メモリ内で

上記の例でメモリ内で何が起こったかを見てみましょう。
`let mut s = String::from("hello");`が実行されると、メモリは次のようになります：

```text
                    s
      +---------+--------+----------+
Stack | pointer | length | capacity | 
      |  |      |   5    |    5     |
      +--|------+--------+----------+
         |
         |
         v
       +---+---+---+---+---+
Heap:  | H | e | l | l | o |
       +---+---+---+---+---+
```

`let t = s.clone()`が実行されると、データのコピーを格納するために完全に新しい領域がヒープに割り当てられます：

```text
                    s                                    t
      +---------+--------+----------+      +---------+--------+----------+
Stack | pointer | length | capacity |      | pointer | length | capacity |
      |  |      |   5    |    5     |      |  |      |   5    |    5     |
      +--|------+--------+----------+      +--|------+--------+----------+
         |                                    |
         |                                    |
         v                                    v
       +---+---+---+---+---+                +---+---+---+---+---+
Heap:  | H | e | l | l | o |                | H | e | l | l | o |
       +---+---+---+---+---+                +---+---+---+---+---+
```

Javaのような言語から来ている場合、`clone`をオブジェクトの深いコピーを作成する方法と考えることができます。

## `Clone`の実装

型を`Clone`可能にするために、その型に対して`Clone`トレイトを実装する必要があります。\
ほぼ常に、deriveすることで`Clone`を実装します：

```rust
#[derive(Clone)]
struct MyType {
    // フィールド
}
```

コンパイラは期待通りに`MyType`に対して`Clone`を実装します：`MyType`の各フィールドを
個別にクローンし、クローンされたフィールドを使用して新しい`MyType`インスタンスを構築します。\
deriveマクロによって生成されたコードを探索するために`cargo expand`（またはIDE）を使用できることを覚えておいてください。