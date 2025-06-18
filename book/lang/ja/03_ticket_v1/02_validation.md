# バリデーション

チケットの定義に戻りましょう：

```rust
struct Ticket {
    title: String,
    description: String,
    status: String,
}
```

`Ticket`構造体のフィールドに「生の」型を使用しています。
これは、ユーザーが空のタイトル、非常に長い説明、またはナンセンスなステータス（例：「Funny」）でチケットを作成できることを意味します。  
もっと良くできます！

## さらなる読み物

- [`String`のドキュメント](https://doc.rust-lang.org/std/string/struct.String.html)をチェックして、提供されているメソッドの詳細な概要を確認してください。演習に必要になります！