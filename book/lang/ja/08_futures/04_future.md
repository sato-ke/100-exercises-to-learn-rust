# `Future`トレイト

## ローカル`Rc`問題

`tokio::spawn`のシグネチャに戻りましょう：

```rust
pub fn spawn<F>(future: F) -> JoinHandle<F::Output>
    where
        F: Future + Send + 'static,
        F::Output: Send + 'static,
{ /* */ }
```

`F`が`Send`であることは_実際に_何を意味するのでしょうか？\
前のセクションで見たように、スポーン環境からキャプチャする値はすべて`Send`でなければ
ならないことを意味します。しかし、それ以上のことがあります。

_.awaitポイントをまたいで保持される_値はすべて`Send`でなければなりません。\
例を見てみましょう：

```rust
use std::rc::Rc;
use tokio::task::yield_now;

fn spawner() {
    tokio::spawn(example());
}

async fn example() {
    // `Send`ではない値、
    // 非同期関数の_内部_で作成される
    let non_send = Rc::new(1);
    
    // 何もしない`.await`ポイント
    yield_now().await;

    // `.await`の後でもローカルの非`Send`値が
    // 必要
    println!("{}", non_send);
}
```

コンパイラはこのコードを拒否します：

```text
error: future cannot be sent between threads safely
    |
5   |     tokio::spawn(example());
    |                  ^^^^^^^^^ 
    | future returned by `example` is not `Send`
    |
note: future is not `Send` as this value is used across an await
    |
11  |     let non_send = Rc::new(1);
    |         -------- has type `Rc<i32>` which is not `Send`
12  |     // A `.await` point
13  |     yield_now().await;
    |                 ^^^^^ 
    |   await occurs here, with `non_send` maybe used later
note: required by a bound in `tokio::spawn`
    |
164 |     pub fn spawn<F>(future: F) -> JoinHandle<F::Output>
    |            ----- required by a bound in this function
165 |     where
166 |         F: Future + Send + 'static,
    |                     ^^^^ required by this bound in `spawn`
```

なぜそうなるのかを理解するには、Rustの非同期モデルの理解を洗練する必要があります。

## `Future`トレイト

`async`関数は**フューチャー**（`Future`トレイトを実装する型）を返すと述べました。
フューチャーは**ステートマシン**と考えることができます。
2つの状態のいずれかにあります：

- **pending**：計算はまだ終了していません。
- **ready**：計算が終了し、出力があります。

これはトレイトの定義でエンコードされています：

```rust
trait Future {
    type Output;
    
    // 今は`Pin`と`Context`を無視してください
    fn poll(
      self: Pin<&mut Self>, 
      cx: &mut Context<'_>
    ) -> Poll<Self::Output>;
}
```

### `poll`

`poll`メソッドは`Future`トレイトの心臓部です。\
フューチャー自体は何もしません。進行するには**ポーリング**される必要があります。\
`poll`を呼び出すと、フューチャーに作業をするよう求めています。
`poll`は進行を試み、次のいずれかを返します：

- `Poll::Pending`：フューチャーはまだ準備ができていません。後で再度`poll`を呼び出す必要があります。
- `Poll::Ready(value)`：フューチャーが終了しました。`value`は計算の結果で、
  型は`Self::Output`です。

`Future::poll`が`Poll::Ready`を返したら、再度ポーリングすべきではありません：
フューチャーは完了し、やることは何も残っていません。

### ランタイムの役割

直接`poll`を呼び出すことはほとんどありません。\
それは非同期ランタイムの仕事です：フューチャーが可能な限り進行できるように、
必要なすべての情報（`poll`のシグネチャの`Context`）を持っています。

## `async fn`とフューチャー

高レベルのインターフェース、非同期関数で作業してきました。\
今、低レベルのプリミティブ、`Future`トレイトを見ました。

それらはどのように関連しているのでしょうか？

関数を非同期としてマークするたびに、その関数はフューチャーを返します。
コンパイラは非同期関数の本体を**ステートマシン**に変換します：
各`.await`ポイントごとに1つの状態です。

`Rc`の例に戻ります：

```rust
use std::rc::Rc;
use tokio::task::yield_now;

async fn example() {
    let non_send = Rc::new(1);
    yield_now().await;
    println!("{}", non_send);
}
```

コンパイラはこれを次のような列挙型に変換します：

```rust
pub enum ExampleFuture {
    NotStarted,
    YieldNow(Rc<i32>),
    Terminated,
}
```

`example`が呼び出されると、`ExampleFuture::NotStarted`を返します。フューチャーは
まだポーリングされていないため、何も起こっていません。\
ランタイムが初めてポーリングすると、`ExampleFuture`は次の`.await`ポイントまで進みます：
ステートマシンの`ExampleFuture::YieldNow(Rc<i32>)`段階で停止し、`Poll::Pending`を返します。\
再度ポーリングされると、残りのコード（`println!`）を実行し、
`Poll::Ready(())`を返します。

ステートマシン表現`ExampleFuture`を見ると、`example`が`Send`ではない理由が
明確になります：`Rc`を保持しているため、`Send`にはなれません。

## イールドポイント

`example`で見たように、すべての`.await`ポイントはフューチャーのライフサイクルに
新しい中間状態を作成します。\
そのため、`.await`ポイントは**イールドポイント**とも呼ばれます：フューチャーは
ポーリングしていたランタイムに_制御を譲る_ことで、ランタイムがそれを一時停止し、
（必要に応じて）別のタスクを実行するようスケジュールできるようにし、
複数の面で同時に進行できるようにします。

後のセクションでイールドの重要性に戻ります。