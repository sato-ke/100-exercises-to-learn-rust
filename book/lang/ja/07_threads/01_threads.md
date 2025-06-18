# スレッド

マルチスレッドコードを書き始める前に、一歩下がって、スレッドとは何か、なぜそれを使いたいのかについて話しましょう。

## スレッドとは何ですか？

**スレッド**は、基盤となるオペレーティングシステムによって管理される実行コンテキストです。\
各スレッドには独自のスタックと命令ポインタがあります。

単一の**プロセス**は複数のスレッドを管理できます。
これらのスレッドは同じメモリ空間を共有するため、同じデータにアクセスできます。

スレッドは**論理的**な構造です。最終的には、CPUコア（**物理的**な実行単位）で一度に一つの命令セットしか実行できません。\
CPUコアよりもはるかに多くのスレッドが存在する可能性があるため、オペレーティングシステムの
**スケジューラ**が、いつどのスレッドを実行するかを決定し、
スループットと応答性を最大化するためにCPU時間を分割します。

## `main`

Rustプログラムが開始されると、単一のスレッド、**メインスレッド**で実行されます。\
このスレッドはオペレーティングシステムによって作成され、`main`
関数を実行する責任があります。

```rust
use std::thread;
use std::time::Duration;

fn main() {
    loop {
        thread::sleep(Duration::from_secs(2));
        println!("Hello from the main thread!");
    }
}
```

## `std::thread`

Rustの標準ライブラリには、スレッドを作成および管理するための`std::thread`モジュールが提供されています。

### `spawn`

`std::thread::spawn`を使用して新しいスレッドを作成し、その上でコードを実行できます。

例えば：

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        loop {
            thread::sleep(Duration::from_secs(1));
            println!("Hello from a thread!");
        }
    });
    
    loop {
        thread::sleep(Duration::from_secs(2));
        println!("Hello from the main thread!");
    }
}
```

[Rust playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=afedf7062298ca8f5a248bc551062eaa)でこのプログラムを実行すると、
メインスレッドとスポーンされたスレッドが並行して実行されることがわかります。\
各スレッドは他のスレッドとは独立して進行します。

### プロセス終了

メインスレッドが終了すると、全体のプロセスが終了します。\
スポーンされたスレッドは、終了するかメインスレッドが終了するまで実行を続けます。

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        loop {
            thread::sleep(Duration::from_secs(1));
            println!("Hello from a thread!");
        }
    });

    thread::sleep(Duration::from_secs(5));
}
```

上記の例では、「Hello from a thread!」メッセージが約5回印刷されることが期待できます。\
その後、メインスレッドが終了し（`sleep`呼び出しが戻るとき）、スポーンされたスレッドは
全体のプロセスが終了するため終了されます。

### `join`

`spawn`が返す`JoinHandle`の`join`メソッドを呼び出すことで、スポーンされたスレッドが終了するのを待つこともできます。

```rust
use std::thread;
fn main() {
    let handle = thread::spawn(|| {
        println!("Hello from a thread!");
    });

    handle.join().unwrap();
}
```

この例では、メインスレッドは終了する前にスポーンされたスレッドが終了するのを待ちます。\
これにより、2つのスレッド間で**同期**の形式が導入されます：プログラムが終了する前に
「Hello from a thread!」メッセージが印刷されることが保証されます。なぜなら、メインスレッドは
スポーンされたスレッドが終了するまで終了しないからです。