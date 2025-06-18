# 構造体

各チケットについて3つの情報を追跡する必要があります：

- タイトル
- 説明
- ステータス

これらを表現するために[`String`](https://doc.rust-lang.org/std/string/struct.String.html)を使用することから始めましょう。`String`はRustの標準ライブラリで定義されている、[UTF-8エンコード](https://en.wikipedia.org/wiki/UTF-8)されたテキストを表現する型です。

しかし、これら3つの情報を単一のエンティティに**組み合わせる**にはどうすればよいでしょうか？

## `struct`の定義

`struct`は**新しいRust型**を定義します。

```rust
struct Ticket {
    title: String,
    description: String,
    status: String
}
```

構造体は、他のプログラミング言語でクラスやオブジェクトと呼ばれるものに非常に似ています。

## フィールドの定義

新しい型は、他の型を**フィールド**として組み合わせることで構築されます。  
各フィールドには名前と型が必要で、コロン`:`で区切られます。複数のフィールドがある場合は、コンマ`,`で区切られます。

以下の`Configuration`構造体で見られるように、フィールドは同じ型である必要はありません：

```rust
struct Configuration {
   version: u32,
   active: bool
}
```

## インスタンス化

各フィールドの値を指定することで構造体のインスタンスを作成できます：

```rust
// 構文: <構造体名> { <フィールド名>: <値>, ... }
let ticket = Ticket {
    title: "Build a ticket system".into(),
    description: "A Kanban board".into(),
    status: "Open".into()
};
```

## フィールドへのアクセス

`.`演算子を使用して構造体のフィールドにアクセスできます：

```rust
// フィールドアクセス
let x = ticket.description;
```

## メソッド

構造体に**メソッド**を定義することで、構造体に動作を結び付けることができます。  
`Ticket`構造体を例に使用します：

```rust
impl Ticket {
    fn is_open(self) -> bool {
        self.status == "Open"
    }
}

// 構文:
// impl <構造体名> {
//    fn <メソッド名>(<パラメータ>) -> <戻り値の型> {
//        // メソッド本体
//    }
// }
```

メソッドは関数と非常に似ていますが、2つの重要な違いがあります：

1. メソッドは**`impl`ブロック**内で定義されなければなりません
2. メソッドは`self`を最初のパラメータとして使用できます。
   `self`はキーワードで、メソッドが呼び出された構造体のインスタンスを表します。

### `self`

メソッドが`self`を最初のパラメータとして取る場合、**メソッド呼び出し構文**を使用して呼び出すことができます：

```rust
// メソッド呼び出し構文: <インスタンス>.<メソッド名>(<パラメータ>)
let is_open = ticket.is_open();
```

これは[前章](../02_basic_calculator/09_saturating.md)で`u32`値に対して飽和算術演算を実行するために使用したのと同じ呼び出し構文です。

### 静的メソッド

メソッドが`self`を最初のパラメータとして取らない場合、それは**静的メソッド**です。

```rust
struct Configuration {
    version: u32,
    active: bool
}

impl Configuration {
    // `default`は`Configuration`の静的メソッドです
    fn default() -> Configuration {
        Configuration { version: 0, active: false }
    }
}
```

静的メソッドを呼び出す唯一の方法は**関数呼び出し構文**を使用することです：

```rust
// 関数呼び出し構文: <構造体名>::<メソッド名>(<パラメータ>)
let default_config = Configuration::default();
```

### 等価性

`self`を最初のパラメータとして取るメソッドに対しても関数呼び出し構文を使用できます：

```rust
// 関数呼び出し構文:
//   <構造体名>::<メソッド名>(<インスタンス>, <パラメータ>)
let is_open = Ticket::is_open(ticket);
```

関数呼び出し構文では、`ticket`がメソッドの最初のパラメータである`self`として使用されていることが明確になりますが、確実により冗長です。可能な場合はメソッド呼び出し構文を優先してください。