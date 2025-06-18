# 引数位置での`impl Trait`

前のセクションでは、`impl Trait`を使用して名前を指定せずに型を返す方法を見ました。\
同じ構文は**引数位置**でも使用できます：

```rust
fn print_iter(iter: impl Iterator<Item = i32>) {
    for i in iter {
        println!("{}", i);
    }
}
```

`print_iter`は`i32`のイテレータを取り、各要素を出力します。\
**引数位置**で使用される場合、`impl Trait`はトレイト境界を持つジェネリックパラメータと等価です：

```rust
fn print_iter<T>(iter: T) 
where
    T: Iterator<Item = i32>
{
    for i in iter {
        println!("{}", i);
    }
}
```

## 欠点

経験則として、引数位置では`impl Trait`よりもジェネリックを好むようにしてください。\
ジェネリックは、呼び出し元がターボフィッシュ構文（`::<>`）を使用して引数の型を明示的に指定することを可能にし、これは曖昧さの解消に役立ちます。これは`impl Trait`の場合はできません。