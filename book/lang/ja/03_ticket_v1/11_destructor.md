# デストラクタ

ヒープを紹介する際に、割り当てたメモリを解放する責任があると述べました。  
借用チェッカーを紹介する際にも、Rustでメモリを直接管理する必要はほとんどないと述べました。

これら2つの文は最初は矛盾しているように見えるかもしれません。
**スコープ**と**デストラクタ**を紹介することで、それらがどのように組み合わさるかを見てみましょう。

## スコープ

変数の**スコープ**は、その変数が有効、または**生きている**Rustコードの領域です。

変数のスコープは宣言から始まります。
以下のいずれかが発生すると終了します：

1. 変数が宣言されたブロック（つまり`{}`間のコード）が終了する
   ```rust
   fn main() {
      // `x`はまだここではスコープ内にありません
      let y = "Hello".to_string();
      let x = "World".to_string(); // <-- xのスコープはここから始まり...
      let h = "!".to_string(); //   |
   } //  <-------------- ...ここで終了します
   ```
2. 変数の所有権が他の誰か（例：関数や別の変数）に転送される
   ```rust
   fn compute(t: String) {
      // 何かする [...]
   }

   fn main() {
       let s = "Hello".to_string(); // <-- sのスコープはここから始まり...
                   //                    | 
       compute(s); // <------------------- ...ここで終了します
                   //   `s`が`compute`に移動されるため
   }
   ```

## デストラクタ

値の所有者がスコープから外れると、Rustはその**デストラクタ**を呼び出します。  
デストラクタはその値によって使用されたリソース、特に割り当てたメモリをクリーンアップしようとします。

`std::mem::drop`に値を渡すことで、値のデストラクタを手動で呼び出すことができます。  
そのため、Rust開発者が「その値は**ドロップ**された」と言って、値がスコープから外れてデストラクタが呼び出されたことを述べるのをよく聞くでしょう。

### ドロップポイントの可視化

コンパイラが私たちのために行うことを「はっきりと」言うために、明示的な`drop`の呼び出しを挿入できます。前の例に戻ります：

```rust
fn main() {
   let y = "Hello".to_string();
   let x = "World".to_string();
   let h = "!".to_string();
}
```

これは以下と同等です：

```rust
fn main() {
   let y = "Hello".to_string();
   let x = "World".to_string();
   let h = "!".to_string();
   // 変数は宣言の逆順でドロップされます
   drop(h);
   drop(x);
   drop(y);
}
```

代わりに2番目の例を見てみましょう。`s`の所有権が`compute`に転送される場合：

```rust
fn compute(s: String) {
   // 何かする [...]
}

fn main() {
   let s = "Hello".to_string();
   compute(s);
}
```

これは以下と同等です：

```rust
fn compute(t: String) {
    // 何かする [...]
    drop(t); // <-- `t`がこの点以前にドロップまたは移動されていないと仮定して、
             //     コンパイラはここで、スコープから外れるときに
             //     `drop`を呼び出します
}

fn main() {
    let s = "Hello".to_string();
    compute(s);
}
```

違いに注目してください：`main`で`compute`が呼ばれた後`s`はもう有効ではありませんが、`main`には`drop(s)`がありません。
値の所有権を関数に転送するとき、**それをクリーンアップする責任も転送している**のです。

これにより、値のデストラクタが**最大[^leak]1回**呼ばれることが保証され、設計により[二重解放バグ](https://owasp.org/www-community/vulnerabilities/Doubly_freeing_memory)を防止します。

### ドロップ後の使用

ドロップされた後に値を使用しようとするとどうなるでしょうか？

```rust
let x = "Hello".to_string();
drop(x);
println!("{}", x);
```

このコードをコンパイルしようとすると、エラーが発生します：

```rust
error[E0382]: use of moved value: `x`
 --> src/main.rs:4:20
  |
3 |     drop(x);
  |          - value moved here
4 |     println!("{}", x);
  |                    ^ value used here after move
```

Drop は呼び出された値を**消費**します。つまり、呼び出し後に値はもう有効ではありません。  
したがって、コンパイラはそれを使用することを防ぎ、[use-after-freeバグ](https://owasp.org/www-community/vulnerabilities/Using_freed_memory)を回避します。

### 参照のドロップ

変数が参照を含んでいる場合はどうでしょうか？  
例えば：

```rust
let x = 42i32;
let y = &x;
drop(y);
```

`drop(y)`を呼び出すと...何も起こりません。  
実際にこのコードをコンパイルしようとすると、警告が表示されます：

```text
warning: calls to `std::mem::drop` with a reference 
         instead of an owned value does nothing
 --> src/main.rs:4:5
  |
4 |     drop(y);
  |     ^^^^^-^
  |          |
  |          argument has type `&i32`
  |
```

これは前に述べたことに戻ります：デストラクタを一度だけ呼び出したいのです。  
同じ値に対して複数の参照を持つことができます—それらの一つがスコープから外れるときに、それらが指す値のデストラクタを呼び出すとしたら、他の参照はどうなるでしょうか？
それらはもう有効でないメモリ位置を参照することになります：いわゆる[**ダングリングポインタ**](https://en.wikipedia.org/wiki/Dangling_pointer)、[**use-after-freeバグ**](https://owasp.org/www-community/vulnerabilities/Using_freed_memory)の近親者です。
Rustの所有権システムは設計によりこの種のバグを排除します。

[^leak]: Rustはデストラクタが実行されることを保証しません。例えば、明示的に[メモリをリーク](../07_threads/03_leak.md)することを選択した場合は実行されません。