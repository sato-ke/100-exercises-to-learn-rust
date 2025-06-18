# `TryFrom`と`TryInto`

前の章では、**失敗しない**型変換のためのRustの慣用的なインターフェースである[`From`と`Into`トレイト](../04_traits/09_from.md)を見ました。  
しかし、変換が成功することが保証されていない場合はどうでしょうか？

エラーについて十分学んだので、`From`と`Into`の**失敗する可能性がある**対応物である`TryFrom`と`TryInto`について議論できます。

## `TryFrom`と`TryInto`

`TryFrom`と`TryInto`の両方は、`From`と`Into`と同様に、`std::convert`モジュールで定義されています。

```rust
pub trait TryFrom<T>: Sized {
    type Error;
    fn try_from(value: T) -> Result<Self, Self::Error>;
}

pub trait TryInto<T>: Sized {
    type Error;
    fn try_into(self) -> Result<T, Self::Error>;
}
```

`From`/`Into`と`TryFrom`/`TryInto`の主な違いは、後者が`Result`型を返すことです。  
これにより、変換が失敗し、panicする代わりにエラーを返すことができます。

## `Self::Error`

`TryFrom`と`TryInto`の両方に関連する`Error`型があります。  
これにより、各実装が独自のエラー型を指定でき、理想的には試行される変換に最も適したものを使用できます。

`Self::Error`は、トレイト自体で定義された`Error`関連型を参照する方法です。

## 二重性

`From`と`Into`と同様に、`TryFrom`と`TryInto`は二重のトレイトです。  
型に`TryFrom`を実装すると、`TryInto`を無料で取得できます。