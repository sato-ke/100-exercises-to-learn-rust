# カプセル化

モジュールと可視性の基本的な理解ができたので、**カプセル化**に戻りましょう。  
カプセル化は、オブジェクトの内部表現を隠す実践です。オブジェクトの状態にいくつかの**不変条件**を適用するために最も一般的に使用されます。

`Ticket`構造体に戻ります：

```rust
struct Ticket {
    title: String,
    description: String,
    status: String,
}
```

すべてのフィールドが公開されている場合、カプセル化はありません。  
フィールドはいつでも変更でき、その型で許可されている任意の値に設定できると仮定しなければなりません。チケットが空のタイトルや意味のないステータスを持つ可能性を排除することはできません。

より厳しいルールを適用するには、フィールドをプライベートに保つ必要があります[^newtype]。
そして、`Ticket`インスタンスと相互作用するための公開メソッドを提供できます。
これらの公開メソッドは、不変条件を維持する責任を持ちます（例：タイトルは空であってはならない）。

少なくとも一つのフィールドがプライベートの場合、構造体のインスタンス化構文を使用して直接`Ticket`インスタンスを作成することはできなくなります：

```rust
// これは動作しません！
let ticket = Ticket {
    title: "Build a ticket system".into(),
    description: "A Kanban board".into(),
    status: "Open".into()
};
```

これは可視性に関する前の演習で実際に見ました。  
モジュールの外側から構造体の新しいインスタンスを作成するために使用できる一つ以上の公開**コンストラクタ**—つまり静的メソッドまたは関数—を提供する必要があります。  
幸い、すでに一つあります：[前の演習](02_validation.md)で実装した`Ticket::new`です。

## アクセサメソッド

要約すると：

- すべての`Ticket`フィールドはプライベートです
- 作成時に検証ルールを適用する公開コンストラクタ`Ticket::new`を提供します

これは良いスタートですが、十分ではありません：`Ticket`を作成する以外に、それと相互作用する必要もあります。
しかし、フィールドがプライベートの場合、どのようにアクセスできるでしょうか？

**アクセサメソッド**を提供する必要があります。  
アクセサメソッドは、構造体のプライベートフィールド（またはフィールド群）の値を読み込むことを可能にする公開メソッドです。

Rustには、他の言語のように、アクセサメソッドを自動生成する組み込み機能はありません。
自分で書く必要があります—それらは通常のメソッドです。

[^newtype]: または型を洗練する、[後で](../05_ticket_v2/15_outro.md)探求するテクニックです。