# タスクのスポーン

前の演習の解答は次のようになっているはずです：

```rust
pub async fn echo(listener: TcpListener) -> Result<(), anyhow::Error> {
    loop {
        let (mut socket, _) = listener.accept().await?;
        let (mut reader, mut writer) = socket.split();
        tokio::io::copy(&mut reader, &mut writer).await?;
    }
}
```

これは悪くありません！\
2つの着信接続の間に長い時間が経過すると、`echo`関数はアイドル状態になります
（`TcpListener::accept`は非同期関数なので）。これにより、エグゼキュータは
その間に他のタスクを実行できます。

しかし、どうすれば実際に複数のタスクを並行して実行できるでしょうか？\
常に非同期関数を完了まで実行する（`.await`を使用して）場合、
一度に実行されるタスクは1つだけになります。

ここで`tokio::spawn`関数の出番です。

## `tokio::spawn`

`tokio::spawn`を使用すると、**完了を待たずに**タスクをエグゼキュータに渡すことができます。\
`tokio::spawn`を呼び出すたびに、スポーンされたタスクをバックグラウンドで、
それをスポーンしたタスクと**並行して**実行し続けるよう`tokio`に指示しています。

複数の接続を並行して処理する方法は次のとおりです：

```rust
use tokio::net::TcpListener;

pub async fn echo(listener: TcpListener) -> Result<(), anyhow::Error> {
    loop {
        let (mut socket, _) = listener.accept().await?;
        // 接続を処理するバックグラウンドタスクをスポーンし、
        // メインタスクがすぐに新しい接続の受け入れを
        // 開始できるようにする
        tokio::spawn(async move {
            let (mut reader, mut writer) = socket.split();
            tokio::io::copy(&mut reader, &mut writer).await?;
        });
    }
}
```

### 非同期ブロック

この例では、`tokio::spawn`に**非同期ブロック**を渡しています：`async move { /* */ }`
非同期ブロックは、別の非同期関数を定義することなく、コードの領域を非同期として
マークする簡単な方法です。

### `JoinHandle`

`tokio::spawn`は`JoinHandle`を返します。\
スポーンされたスレッドに`join`を使用したのと同じ方法で、
`JoinHandle`を使用してバックグラウンドタスクを`.await`できます。

```rust
pub async fn run() {
    // テレメトリデータをリモートサーバーに送信する
    // バックグラウンドタスクをスポーン
    let handle = tokio::spawn(emit_telemetry());
    // その間、他の有用な作業を行う
    do_work().await;
    // ただし、テレメトリデータが正常に配信されるまで
    // 呼び出し元に戻らない
    handle.await;
}

pub async fn emit_telemetry() {
    // [...]
}

pub async fn do_work() {
    // [...]
}
```

### パニック境界

`tokio::spawn`でスポーンされたタスクがパニックすると、パニックはエグゼキュータによってキャッチされます。\
対応する`JoinHandle`を`.await`しない場合、パニックはスポーナーに伝播されません。
`JoinHandle`を`.await`しても、パニックは自動的に伝播されません。
`JoinHandle`を待機すると`Result`が返され、エラー型として[`JoinError`](https://docs.rs/tokio/latest/tokio/task/struct.JoinError.html)
が使用されます。その後、`JoinError::is_panic`を呼び出してタスクがパニックしたかどうかを確認し、
パニックをどう処理するか（ログに記録する、無視する、または伝播する）を選択できます。

```rust
use tokio::task::JoinError;

pub async fn run() {
    let handle = tokio::spawn(work());
    if let Err(e) = handle.await {
        if let Ok(reason) = e.try_into_panic() {
            // タスクがパニックした
            // パニックの巻き戻しを再開し、
            // 現在のタスクに伝播する
            panic::resume_unwind(reason);
        }
    }
}

pub async fn work() {
    // [...]
}
```

### `std::thread::spawn` vs `tokio::spawn`

`tokio::spawn`は`std::thread::spawn`の非同期版の兄弟と考えることができます。

重要な違いに注目してください：`std::thread::spawn`では、OSスケジューラに制御を委任しています。
スレッドがどのようにスケジュールされるかは制御できません。

`tokio::spawn`では、完全にユーザー空間で実行される非同期エグゼキュータに委任しています。
次にどのタスクを実行するかの決定に、基盤となるOSスケジューラは関与しません。
使用することを選択したエグゼキュータを介して、その決定を行う責任は私たちにあります。