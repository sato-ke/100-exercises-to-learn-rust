# 変換、パート1

Rustは整数に対して暗黙的な型変換を実行しないことを何度も繰り返してきました。
では、_明示的な_変換はどのように実行するのでしょうか？

## `as`

`as`演算子を使用して整数型間で変換できます。
`as`変換は**失敗しません**。

例えば：

```rust
let a: u32 = 10;

// `a`を`u64`型にキャストする
let b = a as u64;

// コンパイラによって正しく推論できる場合は、
// ターゲット型として`_`を使用できます。
// 例えば：
let c: u64 = a as _;
```

この変換のセマンティクスは期待通りです：すべての`u32`値は有効な`u64`値です。

### 切り捨て

逆方向に進むと、より興味深いことが起こります：

```rust
// `u8`に収まらないほど大きな数値
let a: u16 = 255 + 1;
let b = a as u8;
```

`as`変換は失敗しないため、このプログラムは問題なく実行されます。
しかし、`b`の値は何でしょうか？
より大きな整数型からより小さな整数型に移行する場合、Rustコンパイラは**切り捨て**を実行します。

何が起こるかを理解するために、まず`256u16`がメモリ内でどのように表現されているか、
ビットのシーケンスとして見てみましょう：

```text
 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0
|               |               |
+---------------+---------------+
  最初の8ビット    最後の8ビット
```

`u8`に変換するとき、Rustコンパイラは`u16`メモリ表現の最後の8ビットを保持します：

```text
 0 0 0 0 0 0 0 0 
|               |
+---------------+
  最後の8ビット
```

したがって、`256 as u8`は`0`に等しくなります。それは...ほとんどのシナリオでは理想的ではありません。
実際、Rustコンパイラは、切り捨てにつながるリテラル値をキャストしようとしているのを見ると、
積極的に止めようとします：

```text
error: literal out of range for `i8`
  |
4 |     let a = 255 as i8;
  |             ^^^
  |
  = note: the literal `255` does not fit into the type `i8` 
          whose range is `-128..=127`
  = help: consider using the type `u8` instead
  = note: `#[deny(overflowing_literals)]` on by default
```

### 推奨事項

経験則として、`as`キャストには十分注意してください。
より小さな型からより大きな型への移行に_専ら_使用してください。
より大きな整数型からより小さな整数型に変換するには、コースの後半で探求する
[_失敗可能な_変換機構](../05_ticket_v2/13_try_from.md)に頼ってください。

### 制限事項

驚くべき動作は`as`キャストの唯一の欠点ではありません。
また、かなり制限されています：プリミティブ型といくつかの特別なケースでのみ`as`キャストに頼ることができます。
複合型を扱う場合は、後で探求する異なる変換メカニズム（[失敗可能](../05_ticket_v2/13_try_from.md)
および[失敗不可能](../04_traits/09_from.md)）に頼る必要があります。

## さらに読む

- 各ソース/ターゲットの組み合わせに対する`as`キャストの正確な動作、
  および許可される変換の網羅的なリストを学ぶには、[Rustの公式リファレンス](https://doc.rust-lang.org/reference/expressions/operator-expr.html#numeric-cast)をチェックしてください。