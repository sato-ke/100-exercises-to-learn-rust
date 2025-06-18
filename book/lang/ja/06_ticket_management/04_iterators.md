# 反復

最初の演習において、Rustでは`for`ループを使用してコレクションを反復できることを学びました。
当時は範囲（例：`0..5`）を見ていましたが、これは配列やベクタなどのコレクションでも同様に当てはまります。

```rust
// `Vec`で動作します
let v = vec![1, 2, 3];
for n in v {
    println!("{}", n);
}

// 配列でも動作します
let a: [u32; 3] = [1, 2, 3];
for n in a {
    println!("{}", n);
}
```

これがどのように内部で動作するかを理解する時が来ました。

## `for`の脱糖

Rustで`for`ループを書くたびに、コンパイラはそれを以下のコードに_脱糖_します：

```rust
let mut iter = IntoIterator::into_iter(v);
loop {
    match iter.next() {
        Some(n) => {
            println!("{}", n);
        }
        None => break,
    }
}
```

`loop`は、`for`と`while`に加えて、もう一つのループ構造です。\
`loop`ブロックは、明示的に`break`で抜け出すまで永続的に実行されます。

## `Iterator`トレイト

前のコードスニペットの`next`メソッドは`Iterator`トレイトから来ています。
`Iterator`トレイトはRustの標準ライブラリで定義されており、値のシーケンスを生成できる型の共通インターフェースを提供します：

```rust
trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}
```

`Item`関連型は、イテレータによって生成される値の型を指定します。

`next`はシーケンス内の次の値を返します。\
返す値がある場合は`Some(value)`を返し、ない場合は`None`を返します。

注意：イテレータが`None`を返したときに枯渇したという保証はありません。これは、イテレータが（より制限的な）[`FusedIterator`](https://doc.rust-lang.org/std/iter/trait.FusedIterator.html)トレイトを実装している場合にのみ保証されます。

## `IntoIterator`トレイト

すべての型が`Iterator`を実装しているわけではありませんが、多くの型は`Iterator`を実装する型に変換できます。\
そこで`IntoIterator`トレイトの出番です：

```rust
trait IntoIterator {
    type Item;
    type IntoIter: Iterator<Item = Self::Item>;
    fn into_iter(self) -> Self::IntoIter;
}
```

`into_iter`メソッドは元の値を消費し、その要素上のイテレータを返します。\
型は`IntoIterator`の実装を一つだけ持つことができます：`for`が何に脱糖されるかについて曖昧さがあってはなりません。

一つの詳細：`Iterator`を実装するすべての型は、自動的に`IntoIterator`も実装します。
`into_iter`から自分自身を返すだけです！

## 境界チェック

イテレータを使った反復には良い副作用があります：設計上、範囲外にはアクセスできません。\
これにより、Rustは生成された機械語コードから境界チェックを除去でき、反復を高速化します。

つまり、

```rust
let v = vec![1, 2, 3];
for n in v {
    println!("{}", n);
}
```

は通常、以下より高速です：

```rust
let v = vec![1, 2, 3];
for i in 0..v.len() {
    println!("{}", v[i]);
}
```

このルールには例外があります：コンパイラは手動インデックスでも範囲外にアクセスしないことを証明できる場合があり、その場合境界チェックを除去します。しかし一般的に、可能な場合はインデックスよりも反復を好むようにしてください。