# Deriveマクロ

`Ticket`に対する`PartialEq`の実装は少し面倒でしたね。
構造体の各フィールドを手動で比較しなければなりませんでした。

## 分解構文

さらに、実装は脆弱です：構造体定義が変更された場合
（例：新しいフィールドが追加された場合）、`PartialEq`実装の更新を忘れないようにする必要があります。

構造体をそのフィールドに**分解**することで、リスクを軽減できます：

```rust
impl PartialEq for Ticket {
    fn eq(&self, other: &Self) -> bool {
        let Ticket {
            title,
            description,
            status,
        } = self;
        // [...]
    }
}
```

`Ticket`の定義が変更されると、コンパイラはエラーを出し、
分解が網羅的でなくなったと文句を言うでしょう。\
変数の隠蔽を避けるために、構造体フィールドの名前を変更することもできます：

```rust
impl PartialEq for Ticket {
    fn eq(&self, other: &Self) -> bool {
        let Ticket {
            title,
            description,
            status,
        } = self;
        let Ticket {
            title: other_title,
            description: other_description,
            status: other_status,
        } = other;
        // [...]
    }
}
```

分解は、ツールキットに持っておくと便利なパターンですが、
これを行うさらに便利な方法があります：**deriveマクロ**です。

## マクロ

過去の練習でいくつかのマクロに既に遭遇しています：

- テストケースの`assert_eq!`と`assert!`
- コンソールに出力する`println!`

Rustマクロは**コード生成器**です。\
提供された入力に基づいて新しいRustコードを生成し、生成されたコードはプログラムの
残りの部分と一緒にコンパイルされます。いくつかのマクロはRustの標準ライブラリに
組み込まれていますが、独自のマクロを書くこともできます。このコースでは独自のマクロを
作成しませんが、[「さらなる読み物」セクション](#further-reading)で有用な
ポインタを見つけることができます。

### 検査

一部のIDEでは、マクロを展開して生成されたコードを検査できます。それが不可能な場合は、
[`cargo-expand`](https://github.com/dtolnay/cargo-expand)を使用できます。

### Deriveマクロ

**deriveマクロ**は、Rustマクロの特別な種類です。構造体の上に**属性**として指定されます。

```rust
#[derive(PartialEq)]
struct Ticket {
    title: String,
    description: String,
    status: String
}
```

Deriveマクロは、カスタム型に対する一般的な（そして「明白な」）トレイトの実装を自動化するために使われます。
上の例では、`PartialEq`トレイトが`Ticket`に自動的に実装されます。
マクロを展開すると、生成されたコードは手動で書いたものと機能的に等価であることがわかりますが、
読むのは少し面倒です：

```rust
#[automatically_derived]
impl ::core::cmp::PartialEq for Ticket {
    #[inline]
    fn eq(&self, other: &Ticket) -> bool {
        self.title == other.title 
            && self.description == other.description
            && self.status == other.status
    }
}
```

コンパイラは、可能な場合はトレイトをderiveするよう促してくれます。

## さらなる読み物

- [The little book of Rust macros](https://veykril.github.io/tlborm/)
- [Proc macro workshop](https://github.com/dtolnay/proc-macro-workshop)