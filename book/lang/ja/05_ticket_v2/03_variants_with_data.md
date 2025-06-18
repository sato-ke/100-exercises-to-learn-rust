# バリアントはデータを保持できる

```rust
enum Status {
    ToDo,
    InProgress,
    Done,
}
```

私たちの`Status` enumは、通常**C風enum**と呼ばれるものです。  
各バリアントは単純なラベルで、名前付き定数のようなものです。この種のenumは、C、C++、Java、C#、Pythonなど、多くのプログラミング言語で見つけることができます。

しかし、Rust enumはさらに進むことができます。各バリアントに**データを添付**できます。

## バリアント

チケットで現在作業している人の名前を保存したいとしましょう。  
この情報は、チケットが進行中の場合にのみ必要になります。To-Doチケットや完了チケットには存在しません。  
`InProgress`バリアントに`String`フィールドを添付することで、これをモデル化できます：

```rust
enum Status {
    ToDo,
    InProgress {
        assigned_to: String,
    },
    Done,
}
```

`InProgress`は**構造体風バリアント**になりました。  
構文は、実際には構造体を定義する際に使用したものと同じです。enumの中にバリアントとして「インライン」されているだけです。

## バリアントデータへのアクセス

`Status`インスタンスで`assigned_to`にアクセスしようとすると、

```rust
let status: Status = /* */;

// これはコンパイルされません
println!("Assigned to: {}", status.assigned_to);
```

コンパイラが止めてくれます：

```text
error[E0609]: no field `assigned_to` on type `Status`
 --> src/main.rs:5:40
  |
5 |     println!("Assigned to: {}", status.assigned_to);
  |                                        ^^^^^^^^^^^ unknown field
```

`assigned_to`は**バリアント固有**で、すべての`Status`インスタンスで利用できるわけではありません。  
`assigned_to`にアクセスするには、**パターンマッチング**を使用する必要があります：

```rust
match status {
    Status::InProgress { assigned_to } => {
        println!("Assigned to: {}", assigned_to);
    },
    Status::ToDo | Status::Done => {
        println!("ToDo or Done");
    }
}
```

## バインディング

マッチパターン`Status::InProgress { assigned_to }`において、`assigned_to`は**バインディング**です。  
`Status::InProgress`バリアントを**分解**して、`assigned_to`フィールドを`assigned_to`という名前の新しい変数にバインドしています。  
望む場合は、フィールドを異なる変数名にバインドすることもできます：

```rust
match status {
    Status::InProgress { assigned_to: person } => {
        println!("Assigned to: {}", person);
    },
    Status::ToDo | Status::Done => {
        println!("ToDo or Done");
    }
}
```