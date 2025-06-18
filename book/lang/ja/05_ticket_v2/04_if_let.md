# 簡潔な分岐

前の演習での解答は、おそらくこのような見た目でしょう：

```rust
impl Ticket {
    pub fn assigned_to(&self) -> &str {
        match &self.status {
            Status::InProgress { assigned_to } => assigned_to,
            Status::Done | Status::ToDo => {
                panic!(
                    "Only `In-Progress` tickets can be \
                    assigned to someone"
                )
            }
        }
    }
}
```

`Status::InProgress`バリアントだけを気にしています。  
本当に他のすべてのバリアントをマッチさせる必要があるでしょうか？

新しい構文が救いの手を差し伸べます！

## `if let`

`if let`構文は、他のすべてのバリアントを処理することなく、enumの単一のバリアントをマッチさせることができます。

`assigned_to`メソッドを簡略化するために`if let`を使用する方法は次のとおりです：

```rust
impl Ticket {
    pub fn assigned_to(&self) -> &str {
        if let Status::InProgress { assigned_to } = &self.status {
            assigned_to
        } else {
            panic!(
                "Only `In-Progress` tickets can be assigned to someone"
            );
        }
    }
}
```

## `let/else`

`else`分岐が早期リターンを意図している場合（panicは早期リターンとしてカウントされます！）、`let/else`構文を使用できます：

```rust
impl Ticket {
    pub fn assigned_to(&self) -> &str {
        let Status::InProgress { assigned_to } = &self.status else {
            panic!(
                "Only `In-Progress` tickets can be assigned to someone"
            );
        };
        assigned_to
    }
}
```

これにより、「右方向のドリフト」を発生させることなく分解された変数を割り当てることができます。つまり、変数はそれに先行するコードと同じインデントレベルで割り当てられます。

## スタイル

`if let`と`let/else`の両方が慣用的なRust構文です。  
コードの可読性を向上させるために適切に使用してください。ただし、やりすぎないでください：必要な時には常に`match`があります。