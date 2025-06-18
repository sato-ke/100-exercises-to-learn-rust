# チケットID

チケット管理システムについて再び考えてみましょう。\
現在のチケットモデルは次のようになっています：

```rust
pub struct Ticket {
    pub title: TicketTitle,
    pub description: TicketDescription,
    pub status: Status
}
```

ここに一つ不足しているものがあります：チケットを一意に識別する**識別子**です。\
その識別子は各チケットに対して一意である必要があります。これは、新しいチケットが作成されるときに自動的に生成することで保証できます。

## モデルの洗練

IDはどこに保存すべきでしょうか？\
`Ticket`構造体に新しいフィールドを追加することができます：

```rust
pub struct Ticket {
    pub id: TicketId,
    pub title: TicketTitle,
    pub description: TicketDescription,
    pub status: Status
}
```

しかし、チケットを作成する前にIDはわかりません。そのため、最初からそこにあることはできません。\
オプショナルにする必要があります：

```rust
pub struct Ticket {
    pub id: Option<TicketId>,
    pub title: TicketTitle,
    pub description: TicketDescription,
    pub status: Status
}
```

これも理想的ではありません—ストアからチケットを取得するたびに`None`ケースを処理しなければならず、チケットが作成された後はIDが常にそこにあるべきだとわかっているにも関わらずです。

最良の解決策は、2つの異なるチケット**状態**を持つことです。これは2つの別々の型で表現されます：
`TicketDraft`と`Ticket`：

```rust
pub struct TicketDraft {
    pub title: TicketTitle,
    pub description: TicketDescription
}

pub struct Ticket {
    pub id: TicketId,
    pub title: TicketTitle,
    pub description: TicketDescription,
    pub status: Status
}
```

`TicketDraft`はまだ作成されていないチケットです。IDもステータスもありません。\
`Ticket`は作成されたチケットです。IDとステータスがあります。\
`TicketDraft`と`Ticket`の各フィールドは独自の制約を埋め込んでいるため、2つの型間でロジックを重複させる必要はありません。