# Errorトレイト

## エラーレポート

前の演習では、`TitleError`バリアントを分解してエラーメッセージを抽出し、`panic!`マクロに渡す必要がありました。  
これは**エラーレポート**の（初歩的な）例です：エラー型をユーザー、サービスオペレータ、または開発者に表示できる表現に変換することです。

各Rust開発者が独自のエラーレポート戦略を考案することは実用的ではありません：時間の無駄であり、プロジェクト間で適切に組み合わせることができません。  
そこで、Rustは`std::error::Error`トレイトを提供しています。

## `Error`トレイト

`Result`の`Err`バリアントの型に制約はありませんが、`Error`トレイトを実装する型を使用することは良い習慣です。  
`Error`はRustのエラーハンドリングストーリーの要です：

```rust
// `Error`トレイトの少し簡略化された定義
pub trait Error: Debug + Display {}
```

[`From`トレイト](../04_traits/09_from.md#supertrait--subtrait)から`:`構文を覚えているかもしれません。これは**スーパートレイト**を指定するために使用されます。  
`Error`には2つのスーパートレイトがあります：`Debug`と`Display`です。型が`Error`を実装したい場合、`Debug`と`Display`も実装する必要があります。

## `Display`と`Debug`

[前の演習](../04_traits/04_derive.md)で`Debug`トレイトにすでに遭遇しています。これは、アサーションが失敗したときに`assert_eq!`が比較している変数の値を表示するために使用されるトレイトです。

「機械的な」観点から、`Display`と`Debug`は同一です。型を文字列風の表現に変換する方法をエンコードします：

```rust
// `Debug`
pub trait Debug {
    fn fmt(&self, f: &mut Formatter<'_>) -> Result<(), Error>;
}

// `Display`
pub trait Display {
    fn fmt(&self, f: &mut Formatter<'_>) -> Result<(), Error>;
}
```

違いは_目的_にあります：`Display`は「エンドユーザー」向けの表現を返し、`Debug`は開発者やサービスオペレータにより適した低レベルの表現を提供します。  
そのため、`Debug`は`#[derive(Debug)]`属性を使用して自動的に実装できますが、`Display`は**手動実装が必要**です。