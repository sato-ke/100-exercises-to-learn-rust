# `'static`

前の演習でベクタからスライスを借用しようとした場合、
おそらく次のようなコンパイラエラーが発生したでしょう：

```text
error[E0597]: `v` does not live long enough
   |
11 | pub fn sum(v: Vec<i32>) -> i32 {
   |            - binding `v` declared here
...
15 |     let right = &v[split_point..];
   |                  ^ borrowed value does not live long enough
16 |     let left_handle = spawn(move || left.iter().sum::<i32>());
   |                             -------------------------------- 
                     argument requires that `v` is borrowed for `'static`
19 | }
   |  - `v` dropped here while still borrowed
```

`argument requires that v is borrowed for 'static`、これは何を意味するのでしょうか？

`'static`ライフタイムはRustの特別なライフタイムです。\
これは、値がプログラムの全期間にわたって有効であることを意味します。

## デタッチされたスレッド

`thread::spawn`で起動されたスレッドは、それをスポーンしたスレッドよりも**長生き**できます。\
例えば：

```rust
use std::thread;

fn f() {
    thread::spawn(|| {
        thread::spawn(|| {
            loop {
                thread::sleep(std::time::Duration::from_secs(1));
                println!("Hello from the detached thread!");
            }
        });
    });
}
```

この例では、最初にスポーンされたスレッドが
毎秒メッセージを印刷する子スレッドをスポーンします。\
最初のスレッドはその後終了します。それが起こると、
その子スレッドは全体のプロセスが実行されている限り**実行を続けます**。\
Rustの用語では、子スレッドが親を**長生き**したと言います。

## `'static`ライフタイム

スポーンされたスレッドは以下のことができるため：

- それをスポーンしたスレッド（親スレッド）よりも長生きする
- プログラムが終了するまで実行する

プログラムが終了する前にドロップされる可能性のある値を借用してはなりません。
この制約に違反すると、use-after-freeバグにさらされます。\
そのため、`std::thread::spawn`のシグネチャでは、渡されるクロージャが
`'static`ライフタイムを持つことを要求しています：

```rust
pub fn spawn<F, T>(f: F) -> JoinHandle<T> 
where
    F: FnOnce() -> T + Send + 'static,
    T: Send + 'static
{
    // [..]
}
```

## `'static`は参照（だけ）ではありません

Rustのすべての値にはライフタイムがあり、参照だけではありません。

特に、データを所有する型（`Vec`や`String`など）は
`'static`制約を満たします：それを所有していれば、それを最初に作成した
関数が戻った後でも、好きなだけ長く作業を続けることができます。

したがって、`'static`を次のように解釈することができます：

- 所有された値を与えてください
- プログラムの全期間にわたって有効な参照を与えてください

最初のアプローチは、前の演習で問題を解決した方法です：
元のベクタの左右の部分を保持する新しいベクタを割り当て、
それらをスポーンされたスレッドに移動しました。

## `'static`参照

2番目のケース、プログラムの全期間にわたって有効な
参照について話しましょう。

### 静的データ

最も一般的なケースは、文字列リテラルなどの**静的データ**への参照です：

```rust
let s: &'static str = "Hello world!";
```

文字列リテラルはコンパイル時に知られているため、Rustはそれらを実行可能ファイルの_内部_、
**読み取り専用データセグメント**として知られる領域に格納します。
その領域を指すすべての参照は、
プログラムが実行されている限り有効になります。つまり、`'static`契約を満たします。

## 参考文献

- [データセグメント](https://en.wikipedia.org/wiki/Data_segment)