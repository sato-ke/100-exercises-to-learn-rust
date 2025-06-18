# `match`

enumで実際に**何ができるのか**疑問に思うかもしれません。  
最も一般的な操作は、**マッチ**することです。

```rust
enum Status {
    ToDo,
    InProgress,
    Done
}

impl Status {
    fn is_done(&self) -> bool {
        match self {
            Status::Done => true,
            // `|`演算子は複数のパターンをマッチさせることができます。
            // これは「`Status::ToDo`または`Status::InProgress`のいずれか」として読みます。
            Status::InProgress | Status::ToDo => false
        }
    }
}
```

`match`文は、Rustの値を一連の**パターン**と比較できる文です。  
これを型レベルの`if`として考えることができます。`status`が`Done`バリアントの場合、最初のブロックを実行し、`InProgress`または`ToDo`バリアントの場合、2番目のブロックを実行します。

## 網羅性

ここで重要な詳細が一つあります：`match`は**網羅的**です。すべてのenumバリアントを処理する必要があります。  
バリアントの処理を忘れると、Rustは**コンパイル時**にエラーで止めてくれます。

例えば、`ToDo`バリアントの処理を忘れた場合：

```rust
match self {
    Status::Done => true,
    Status::InProgress => false,
}
```

コンパイラは文句を言います：

```text
error[E0004]: non-exhaustive patterns: `ToDo` not covered
 --> src/main.rs:5:9
  |
5 |     match status {
  |     ^^^^^^^^^^^^ pattern `ToDo` not covered
```

これは重要です！  
コードベースは時間とともに進化します。将来的に新しいステータス（例：`Blocked`）を追加するかもしれません。Rustコンパイラは、新しいバリアントのロジックが欠けているすべての`match`文に対してエラーを出力します。  
これが、Rust開発者がしばしば「コンパイラ駆動リファクタリング」を称賛する理由です。コンパイラが次に何をすべきかを教えてくれ、報告された内容を修正するだけで済みます。

## キャッチオール

一つまたは複数のバリアントを気にしない場合、`_`パターンをキャッチオールとして使用できます：

```rust
match status {
    Status::Done => true,
    _ => false
}
```

`_`パターンは、前のパターンでマッチしなかったものすべてにマッチします。

<div class="warning">
このキャッチオールパターンを使用することで、コンパイラ駆動リファクタリングの恩恵を_受けられなく_なります。  
新しいenumバリアントを追加しても、コンパイラはそれを処理していないことを_教えてくれません_。

正確性を重視する場合は、キャッチオールの使用を避けてください。コンパイラを活用してすべてのマッチングサイトを再検討し、新しいenumバリアントをどのように処理すべきかを決定しましょう。

</div>