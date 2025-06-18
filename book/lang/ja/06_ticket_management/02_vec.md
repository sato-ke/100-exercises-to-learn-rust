# ベクタ

配列の強みは同時に弱みでもあります：サイズは事前に、コンパイル時に分かっている必要があります。
ランタイムでのみ分かるサイズで配列を作成しようとすると、コンパイルエラーが発生します：

```rust
let n = 10;
let numbers: [u32; n];
```

```text
error[E0435]: attempt to use a non-constant value in a constant
 --> src/main.rs:3:20
  |
2 | let n = 10;
3 | let numbers: [u32; n];
  |                    ^ non-constant value
```

配列は我々のチケット管理システムには適しません—コンパイル時に何個のチケットを保存する必要があるかわからないからです。
ここで`Vec`の出番です。

## `Vec`

`Vec`は、標準ライブラリによって提供される成長可能な配列型です。\
`Vec::new`関数を使用して空の配列を作成できます：

```rust
let mut numbers: Vec<u32> = Vec::new();
```

その後、`push`メソッドを使用してベクタに要素をプッシュします：

```rust
numbers.push(1);
numbers.push(2);
numbers.push(3);
```

新しい値はベクタの末尾に追加されます。\
作成時に値がわかっている場合は、`vec!`マクロを使用して初期化されたベクタを作成することもできます：

```rust
let numbers = vec![1, 2, 3];
```

## 要素へのアクセス

要素にアクセスする構文は配列と同じです：

```rust
let numbers = vec![1, 2, 3];
let first = numbers[0];
let second = numbers[1];
let third = numbers[2];
```

インデックスは`usize`型である必要があります。\
`get`メソッドも使用できます。これは`Option<&T>`を返します：

```rust
let numbers = vec![1, 2, 3];
assert_eq!(numbers.get(0), Some(&1));
// 範囲外のインデックスにアクセスしようとすると、
// パニックの代わりに`None`が得られます。
assert_eq!(numbers.get(3), None);
```

アクセスは、配列の要素アクセスと同様に境界チェックされます。計算量は`O(1)`です。

## メモリレイアウト

`Vec`はヒープ割り当てのデータ構造です。\
`Vec`を作成すると、要素を保存するためのメモリをヒープに割り当てます。

以下のコードを実行すると：

```rust
let mut numbers = Vec::with_capacity(3);
numbers.push(1);
numbers.push(2);
```

次のメモリレイアウトが得られます：

```text
      +---------+--------+----------+
Stack | pointer | length | capacity | 
      |  |      |   2    |    3     |
      +--|------+--------+----------+
         |
         |
         v
       +---+---+---+
Heap:  | 1 | 2 | ? |
       +---+---+---+
```

`Vec`は3つのことを追跡します：

- ヒープ領域に予約した**ポインタ**。
- ベクタの**長さ**、すなわちベクタ内にある要素数。
- ベクタの**容量**、すなわちヒープ上に予約されたスペースに収まる要素数。

このレイアウトは見覚えがあるはずです：`String`とまったく同じです！\
これは偶然ではありません：`String`は内部的にバイトのベクタ、`Vec<u8>`として定義されています：

```rust
pub struct String {
    vec: Vec<u8>,
}
```