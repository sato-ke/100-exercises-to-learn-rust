# スコープスレッド

これまでに議論したライフタイムの問題はすべて共通の原因があります：
スポーンされたスレッドが親よりも長生きできることです。\
**スコープスレッド**を使用することで、この問題を回避できます。

```rust
let v = vec![1, 2, 3];
let midpoint = v.len() / 2;

std::thread::scope(|scope| {
    scope.spawn(|| {
        let first = &v[..midpoint];
        println!("Here's the first half of v: {first:?}");
    });
    scope.spawn(|| {
        let second = &v[midpoint..];
        println!("Here's the second half of v: {second:?}");
    });
});

println!("Here's v: {v:?}");
```

何が起こっているかを分解してみましょう。

## `scope`

`std::thread::scope`関数は新しい**スコープ**を作成します。\
`std::thread::scope`は入力としてクロージャを受け取り、単一の引数：`Scope`インスタンスを持ちます。

## スコープスポーン

`Scope`は`spawn`メソッドを公開します。\
`std::thread::spawn`とは異なり、`Scope`を使用してスポーンされたすべてのスレッドは、
スコープが終了するときに**自動的にジョイン**されます。

前の例を`std::thread::spawn`に「翻訳」すると、
次のようになります：

```rust
let v = vec![1, 2, 3];
let midpoint = v.len() / 2;

let handle1 = std::thread::spawn(|| {
    let first = &v[..midpoint];
    println!("Here's the first half of v: {first:?}");
});
let handle2 = std::thread::spawn(|| {
    let second = &v[midpoint..];
    println!("Here's the second half of v: {second:?}");
});

handle1.join().unwrap();
handle2.join().unwrap();

println!("Here's v: {v:?}");
```

## 環境からの借用

ただし、翻訳された例はコンパイルされません：コンパイラは
`&v`のライフタイムが`'static`ではないため、スポーンされたスレッドから使用できないと文句を言うでしょう。

`std::thread::scope`ではこれは問題ありません—**環境から安全に借用**できます。

この例では、`v`はスポーンポイントの前に作成されます。
`scope`が戻った_後_にのみドロップされます。同時に、
`scope`内でスポーンされたすべてのスレッドは、`scope`が戻る_前_に終了することが保証されているため、
ダングリング参照のリスクはありません。

コンパイラは文句を言いません！