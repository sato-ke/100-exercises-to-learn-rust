# アンラップ

`Ticket::new`は無効な入力でpanicする代わりに`Result`を返すようになりました。  
これは呼び出し元にとって何を意味するでしょうか？

## 失敗は（暗黙的に）無視できない

例外とは異なり、RustのResult`は**呼び出し場所でエラーを処理する**ことを強制します。  
`Result`を返す関数を呼び出す場合、Rustはエラーケースを暗黙的に無視することを許可しません。

```rust
fn parse_int(s: &str) -> Result<i32, ParseIntError> {
    // ...
}

// これはコンパイルされません：エラーケースを処理していません。
// `match`またはResultが提供するコンビネータの一つを使用して
// 成功値を「アンラップ」するか、エラーを処理する必要があります。
let number = parse_int("42") + 2;
```

## `Result`を受け取りました。さて、どうしますか？

`Result`を返す関数を呼び出すとき、2つの主要なオプションがあります：

- 操作が失敗した場合にpanicします。  
  これは`unwrap`または`expect`メソッドのいずれかを使用して行われます。
  ```rust
  // `parse_int`が`Err`を返すとpanicします。
  let number = parse_int("42").unwrap();
  // `expect`はカスタムpanicメッセージを指定できます。
  let number = parse_int("42").expect("Failed to parse integer");
  ```
- `match`式を使用して`Result`を分解し、エラーケースを明示的に処理します。
  ```rust
  match parse_int("42") {
      Ok(number) => println!("Parsed number: {}", number),
      Err(err) => eprintln!("Error: {}", err),
  }
  ```