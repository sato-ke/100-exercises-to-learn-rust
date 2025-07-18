# ケースバイケースの動作

`overflow-checks`は鈍いツールです：プログラム全体に影響を与えるグローバル設定です。
コンテキストに応じて整数オーバーフローを異なる方法で処理したい場合がよくあります：時にはラッピングが正しい選択であり、
他の時にはパニックが望ましいです。

## `wrapping_`メソッド

`wrapping_`メソッド[^method]を使用して、操作ごとにラッピング算術を選択できます。
例えば、`wrapping_add`を使用して2つの整数をラッピングで加算できます：

```rust
let x = 255u8;
let y = 1u8;
let sum = x.wrapping_add(y);
assert_eq!(sum, 0);
```

## `saturating_`メソッド

代わりに、`saturating_`メソッドを使用して**飽和算術**を選択できます。
ラップアラウンドする代わりに、飽和算術は整数型の最大値または最小値を返します。
例えば：

```rust
let x = 255u8;
let y = 1u8;
let sum = x.saturating_add(y);
assert_eq!(sum, 255);
```

`255 + 1`は`256`で、`u8::MAX`より大きいため、結果は`u8::MAX`（255）になります。
アンダーフローでは逆のことが起こります：`0 - 1`は`-1`で、`u8::MIN`より小さいため、結果は`u8::MIN`（0）になります。

`overflow-checks`プロファイル設定を介して飽和算術を取得することはできません—算術演算を実行するときに明示的にオプトインする必要があります。

[^method]: メソッドは特定の型に「接続された」関数と考えることができます。
次の章でメソッド（およびそれらの定義方法）について説明します。