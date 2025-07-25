# ランタイムをブロックしない

イールドポイントに戻りましょう。\
スレッドとは異なり、**Rustのタスクはプリエンプトできません**。

`tokio`は独自の判断でタスクを一時停止し、代わりに別のタスクを実行することはできません。
制御がエグゼキュータに戻るのは**排他的に**タスクがイールドするとき、つまり
`Future::poll`が`Poll::Pending`を返すとき、または`async fn`の場合、
フューチャーを`.await`するときです。

これはランタイムをリスクにさらします：タスクが決してイールドしない場合、
ランタイムは別のタスクを実行できません。これを**ランタイムのブロッキング**と呼びます。

## ブロッキングとは？

どのくらいが長すぎるのでしょうか？タスクが問題になる前に、イールドせずに
どれだけの時間を費やすことができるでしょうか？

ランタイム、アプリケーション、実行中のタスク数、その他多くの要因に依存します。
しかし、一般的な経験則として、イールドポイント間で100マイクロ秒未満に
抑えるようにしてください。

## 結果

ランタイムのブロッキングは以下につながる可能性があります：

- **デッドロック**：イールドしないタスクが別のタスクの完了を待っており、
  そのタスクが最初のタスクのイールドを待っている場合、デッドロックが発生します。
  ランタイムが別のスレッドでタスクをスケジュールできない限り、進行できません。
- **スターベーション**：他のタスクが実行できない、または長い遅延の後に実行される
  可能性があり、パフォーマンスの低下（例：高いテイルレイテンシー）につながる可能性があります。

## ブロッキングは常に明白ではない

一般的に非同期コードで避けるべき操作のタイプがあります：

- 同期I/O。どれくらい時間がかかるか予測できず、100マイクロ秒を超える可能性が高いです。
- 高価なCPUバウンドな計算。

後者のカテゴリは常に明白ではありません。例えば、少数の要素を持つベクターの
ソートは問題ありません。ベクターに数十億のエントリがある場合、その評価は変わります。

## ブロッキングを避ける方法

OK、ブロッキングと見なされる、またはブロッキングと見なされるリスクのある操作を
_実行しなければならない_場合、どうやってランタイムのブロッキングを避けるのでしょうか？\
作業を別のスレッドに移動する必要があります。`tokio`がタスクを実行するために使用する、
いわゆるランタイムスレッドは使用したくありません。

`tokio`はこの目的のために**ブロッキングプール**と呼ばれる専用のスレッドプールを提供します。
`tokio::task::spawn_blocking`関数を使用して、ブロッキングプール上で同期操作を
スポーンできます。`spawn_blocking`は、操作が完了したときにその結果に解決される
フューチャーを返します。

```rust
use tokio::task;

fn expensive_computation() -> u64 {
    // [...]
}

async fn run() {
    let handle = task::spawn_blocking(expensive_computation);
    // その間に他のことをする
    let result = handle.await.unwrap();
}
```

ブロッキングプールは長寿命です。`spawn_blocking`は、スレッド初期化のコストが
複数の呼び出しにわたって償却されるため、`std::thread::spawn`を介して
直接新しいスレッドを作成するよりも高速である必要があります。

## さらに読む

- このトピックに関する[Alice Ryhlのブログ投稿](https://ryhl.io/blog/async-what-is-blocking/)
  をチェックしてください。