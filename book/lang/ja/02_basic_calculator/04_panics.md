# パニック

["変数" セクション](02_variables.md)で書いた`speed`関数に戻りましょう。
おそらく次のようなものだったでしょう：

```rust
fn speed(start: u32, end: u32, time_elapsed: u32) -> u32 {
    let distance = end - start;
    distance / time_elapsed
}
```

鋭い観察力があれば、1つの問題[^one]に気づいたかもしれません：`time_elapsed`がゼロの場合、何が起こるでしょうか？

[Rustプレイグラウンド](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=36e5ddbe3b3f741dfa9f74c956622bac)で試すことができます！\
プログラムは次のエラーメッセージとともに終了します：

```text
thread 'main' panicked at src/main.rs:3:5:
attempt to divide by zero
```

これは**パニック**と呼ばれます。\
パニックは、プログラムが続行できないほど何かが間違っていることを示すRustの方法であり、**回復不可能なエラー**[^catching]です。ゼロ除算はそのようなエラーとして分類されます。

## panic!マクロ

`panic!`マクロ[^macro]を呼び出すことで、意図的にパニックを引き起こすことができます：

```rust
fn main() {
    panic!("This is a panic!");
    // 以下の行は実行されません
    let x = 1 + 2;
}
```

Rustには回復可能なエラーを扱う他のメカニズムがあり、[後で説明します](../05_ticket_v2/06_fallibility.md)。
当面は、粗暴ですが単純な応急処置としてパニックを使用します。

## さらに読む

- [panic!マクロのドキュメント](https://doc.rust-lang.org/std/macro.panic.html)

[^one]: `speed`にはもう1つ問題があり、すぐに対処します。それを見つけられますか？

[^catching]: パニックをキャッチしようとすることはできますが、非常に特定の状況に限定された最後の手段であるべきです。

[^macro]: `!`が続く場合、それはマクロ呼び出しです。今のところマクロをスパイシーな関数と考えてください。コースの後半で詳しく説明します。