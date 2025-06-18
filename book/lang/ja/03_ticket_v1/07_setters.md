# 可変参照

アクセサメソッドは今このようになっているはずです：

```rust
impl Ticket {
    pub fn title(&self) -> &String {
        &self.title
    }

    pub fn description(&self) -> &String {
        &self.description
    }

    pub fn status(&self) -> &String {
        &self.status
    }
}
```

あちこちに`&`を少し振りかけることで解決しました！  
プロセスで消費することなく`Ticket`インスタンスのフィールドにアクセスする方法ができました。
次に**セッターメソッド**で`Ticket`構造体を強化する方法を見てみましょう。

## セッター

セッターメソッドを使用すると、ユーザーは`Ticket`のプライベートフィールドの値を変更しながら、その不変条件が尊重されることを確実にできます（つまり、`Ticket`のタイトルを空の文字列に設定することはできません）。

Rustでセッターを実装する一般的な方法は2つあります：

- `self`を入力として取る。
- `&mut self`を入力として取る。

### `self`を入力として取る

最初のアプローチはこのようになります：

```rust
impl Ticket {
    pub fn set_title(mut self, new_title: String) -> Self {
        // 新しいタイトルを検証 [...]
        self.title = new_title;
        self
    }
}
```

`self`の所有権を取得し、タイトルを変更し、変更された`Ticket`インスタンスを返します。  
これを使用する方法：

```rust
let ticket = Ticket::new(
    "Title".into(), 
    "Description".into(), 
    "To-Do".into()
);
let ticket = ticket.set_title("New title".into());
```

`set_title`は`self`の所有権を取得する（つまり**消費する**）ので、結果を変数に再代入する必要があります。
上記の例では、**変数シャドウイング**を利用して同じ変数名を再利用します：既存の変数と同じ名前で新しい変数を宣言すると、新しい変数が古い変数を**シャドウ**します。これはRustコードでは一般的なパターンです。

`self`セッターは、複数のフィールドを一度に変更する必要がある場合に非常にうまく機能します：複数の呼び出しを連鎖できます！

```rust
let ticket = ticket
    .set_title("New title".into())
    .set_description("New description".into())
    .set_status("In Progress".into());
```

### `&mut self`を入力として取る

セッターの2番目のアプローチ、`&mut self`を使用するものは、代わりにこのようになります：

```rust
impl Ticket {
    pub fn set_title(&mut self, new_title: String) {
        // 新しいタイトルを検証 [...]
        
        self.title = new_title;
    }
}
```

今回メソッドは`self`への可変参照を入力として取り、タイトルを変更し、それで終わりです。
何も返されません。

このように使用します：

```rust
let mut ticket = Ticket::new(
    "Title".into(),
    "Description".into(),
    "To-Do".into()
);
ticket.set_title("New title".into());

// 変更されたチケットを使用
```

所有権は呼び出し元に残るので、元の`ticket`変数はまだ有効です。結果を再代入する必要はありません。
ただし、可変参照を取るので、`ticket`を可変としてマークする必要があります。

`&mut`セッターには欠点があります：複数の呼び出しを連鎖できません。
変更された`Ticket`インスタンスを返さないので、最初のセッターの結果で別のセッターを呼び出すことはできません。
各セッターを個別に呼び出す必要があります：

```rust
ticket.set_title("New title".into());
ticket.set_description("New description".into());
ticket.set_status("In Progress".into());
```