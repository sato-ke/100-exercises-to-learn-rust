# キャンセレーション

保留中のフューチャーがドロップされると何が起こるでしょうか？\
ランタイムはもうそれをポーリングしないため、それ以上進行しません。
言い換えれば、その実行は**キャンセル**されました。

実際には、これはタイムアウトを扱うときによく起こります。
例えば：

```rust
use tokio::time::timeout;
use tokio::sync::oneshot;
use std::time::Duration;

async fn http_call() {
    // [...]
}

async fn run() {
    // フューチャーを10ミリ秒で期限切れになる`Timeout`でラップする
    let duration = Duration::from_millis(10);
    if let Err(_) = timeout(duration, http_call()).await {
        println!("Didn't receive a value within 10 ms");
    }
}
```

タイムアウトが期限切れになると、`http_call`によって返されるフューチャーはキャンセルされます。
これが`http_call`の本体だと想像してみましょう：

```rust
use std::net::TcpStream;

async fn http_call() {
    let (stream, _) = TcpStream::connect(/* */).await.unwrap();
    let request: Vec<u8> = /* */;
    stream.write_all(&request).await.unwrap();
}
```

各イールドポイントは**キャンセルポイント**になります。\
`http_call`はランタイムによってプリエンプトできないため、`.await`を介して
エグゼキュータに制御を戻した後にのみ破棄できます。
これは再帰的に適用されます。例えば、`stream.write_all(&request)`は実装に
複数のイールドポイントがある可能性があります。`http_call`がキャンセルされる前に
_部分的な_リクエストをプッシュし、接続をドロップしてボディの送信を
完了しないことは十分に可能です。

## クリーンアップ

Rustのキャンセルメカニズムは非常に強力です。呼び出し元は、タスク自体からの
協力を必要とせずに、進行中のタスクをキャンセルできます。\
同時に、これは非常に危険な場合があります。操作を中止する前に、いくつかの
クリーンアップタスクが実行されることを確保するために、**優雅なキャンセル**を
実行することが望ましい場合があります。

例えば、SQLトランザクション用のこの架空のAPIを考えてみてください：

```rust
async fn transfer_money(
    connection: SqlConnection,
    payer_id: u64,
    payee_id: u64,
    amount: u64
) -> Result<(), anyhow::Error> {
    let transaction = connection.begin_transaction().await?;
    update_balance(payer_id, amount, &transaction).await?;
    decrease_balance(payee_id, amount, &transaction).await?;
    transaction.commit().await?;
}
```

キャンセル時には、保留中のトランザクションをハングさせたままにするのではなく、
明示的に中止することが理想的です。
残念ながら、Rustはこの種の**非同期**クリーンアップ操作のための確実なメカニズムを
提供していません。

最も一般的な戦略は、必要なクリーンアップ作業をスケジュールするために
`Drop`トレイトに依存することです。これは次の方法で行えます：

- ランタイムに新しいタスクをスポーンする
- チャネルにメッセージをエンキューする
- バックグラウンドスレッドをスポーンする

最適な選択は文脈に依存します。

## スポーンされたタスクのキャンセル

`tokio::spawn`を使用してタスクをスポーンすると、もうドロップできません。
それはランタイムに属しています。\
それでも、必要に応じて`JoinHandle`を使用してキャンセルできます：

```rust
async fn run() {
    let handle = tokio::spawn(/* 何らかの非同期タスク */);
    // スポーンされたタスクをキャンセル
    handle.abort();
}
```

## さらに読む

- 2つの異なるフューチャーを「レース」させるために`tokio`の`select!`マクロを使用する場合は
  非常に注意してください。**キャンセル安全性**を確保できない限り、ループで同じタスクを
  再試行することは危険です。詳細については[`select!`のドキュメント](https://tokio.rs/tokio/tutorial/select)を確認してください。\
  2つの非同期データストリーム（例：ソケットとチャネル）をインターリーブする必要がある場合は、
  代わりに[`StreamExt::merge`](https://docs.rs/tokio-stream/latest/tokio_stream/trait.StreamExt.html#method.merge)を使用することを推奨します。
- 場合によっては、[`CancellationToken`](https://docs.rs/tokio-util/latest/tokio_util/sync/struct.CancellationToken.html)が
  `JoinHandle::abort`よりも好ましい場合があります。