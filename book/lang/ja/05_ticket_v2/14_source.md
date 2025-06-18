# `Error::source`

`Error`トレイトの説明を完了するために、もう一つ話すべきことがあります：`source`メソッドです。

```rust
// 今度は完全な定義です！
pub trait Error: Debug + Display {
    fn source(&self) -> Option<&(dyn Error + 'static)> {
        None
    }
}
```

`source`メソッドは、**エラーの原因**にアクセスする方法です（もしあれば）。  
エラーはしばしば連鎖しています。つまり、一つのエラーが別のエラーの原因となります：高レベルエラー（例：データベースに接続できない）があり、それは低レベルエラー（例：データベースのホスト名を解決できない）によって引き起こされます。  
`source`メソッドを使用すると、エラーの完全な連鎖を「歩く」ことができ、ログでエラーコンテキストをキャプチャする際によく使用されます。

## `source`の実装

`Error`トレイトは、常に`None`を返すデフォルト実装を提供します（つまり、根本原因なし）。そのため、前の演習では`source`を気にする必要がありませんでした。  
このデフォルト実装をオーバーライドして、エラー型の原因を提供できます。

```rust
use std::error::Error;

#[derive(Debug)]
struct DatabaseError {
    source: std::io::Error
}

impl std::fmt::Display for DatabaseError {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "Failed to connect to the database")
    }
}

impl std::error::Error for DatabaseError {
    fn source(&self) -> Option<&(dyn Error + 'static)> {
        Some(&self.source)
    }
}
```

この例では、`DatabaseError`は`std::io::Error`をそのソースとしてラップします。  
その後、`source`メソッドをオーバーライドして、呼び出されたときにこのソースを返します。

## `&(dyn Error + 'static)`

この`&(dyn Error + 'static)`型は何でしょうか？  
これを分解してみましょう：

- `dyn Error`は**トレイトオブジェクト**です。`Error`トレイトを実装する任意の型を参照する方法です。
- `'static`は特別な**ライフタイム指定子**です。  
  `'static`は、参照が「必要な限り」、つまりプログラム実行全体にわたって有効であることを意味します。

組み合わせ：`&(dyn Error + 'static)`は、`Error`トレイトを実装し、プログラム実行全体にわたって有効なトレイトオブジェクトへの参照です。

これらの概念については今のところあまり心配しないでください。将来の章でより詳細に扱います。

## `thiserror`を使用した`source`の実装

`thiserror`は、エラー型に`source`を自動的に実装する3つの方法を提供します：

- `source`という名前のフィールドが自動的にエラーのソースとして使用されます。
  ```rust
  use thiserror::Error;

  #[derive(Error, Debug)]
  pub enum MyError {
      #[error("Failed to connect to the database")]
      DatabaseError {
          source: std::io::Error
      }
  }
  ```
- `#[source]`属性で注釈されたフィールドが自動的にエラーのソースとして使用されます。
  ```rust
  use thiserror::Error;

  #[derive(Error, Debug)]
  pub enum MyError {
      #[error("Failed to connect to the database")]
      DatabaseError {
          #[source]
          inner: std::io::Error
      }
  }
  ```
- `#[from]`属性で注釈されたフィールドが自動的にエラーのソースとして使用され、**かつ**`thiserror`が注釈された型をエラー型に変換する`From`実装を自動的に生成します。
  ```rust
  use thiserror::Error;

  #[derive(Error, Debug)]
  pub enum MyError {
      #[error("Failed to connect to the database")]
      DatabaseError {
          #[from]
          inner: std::io::Error
      }
  }
  ```

## `?`演算子

`?`演算子は、エラーを伝播するためのショートハンドです。  
`Result`を返す関数で使用すると、`Result`が`Err`の場合にエラーで早期リターンします。

例えば：

```rust
use std::fs::File;

fn read_file() -> Result<String, std::io::Error> {
    let mut file = File::open("file.txt")?;
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;
    Ok(contents)
}
```

は次と同等です：

```rust
use std::fs::File;

fn read_file() -> Result<String, std::io::Error> {
    let mut file = match File::open("file.txt") {
        Ok(file) => file,
        Err(e) => {
            return Err(e);
        }
    };
    let mut contents = String::new();
    match file.read_to_string(&mut contents) {
        Ok(_) => (),
        Err(e) => {
            return Err(e);
        }
    }
    Ok(contents)
}
```

`?`演算子を使用すると、エラーハンドリングコードを大幅に短縮できます。  
特に、`?`演算子は、変換が可能であれば（つまり、適切な`From`実装がある場合）、失敗可能な操作のエラー型を関数のエラー型に自動的に変換します。