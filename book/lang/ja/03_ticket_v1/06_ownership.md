# 所有権

これまでにこのコースで教わったことを使って前の演習を解いた場合、
アクセサメソッドはおそらくこのようになっているでしょう：

```rust
impl Ticket {
    pub fn title(self) -> String {
        self.title
    }

    pub fn description(self) -> String {
        self.description
    }

    pub fn status(self) -> String {
        self.status
    }
}
```

これらのメソッドはコンパイルされ、テストを通すのに十分ですが、実際のシナリオではあまり役に立ちません。
このスニペットを考えてみてください：

```rust
if ticket.status() == "To-Do" {
    // `println!`マクロはまだ扱っていませんが、
    // 今のところ、これは（テンプレート化された）メッセージを
    // コンソールに印刷することを知っていれば十分です
    println!("Your next task is: {}", ticket.title());
}
```

これをコンパイルしようとすると、エラーが発生します：

```text
error[E0382]: use of moved value: `ticket`
  --> src/main.rs:30:43
   |
25 |     let ticket = Ticket::new(/* */);
   |         ------ move occurs because `ticket` has type `Ticket`, 
   |                which does not implement the `Copy` trait
26 |     if ticket.status() == "To-Do" {
   |               -------- `ticket` moved due to this method call
...
30 |         println!("Your next task is: {}", ticket.title());
   |                                           ^^^^^^ 
   |                                value used here after move
   |
note: `Ticket::status` takes ownership of the receiver `self`, 
      which moves `ticket`
  --> src/main.rs:12:23
   |
12 |         pub fn status(self) -> String {
   |                       ^^^^
```

おめでとうございます、これがあなたの最初の借用チェッカーエラーです！

## Rustの所有権システムの利点

Rustの所有権システムは以下を保証するように設計されています：

- データは読み込まれている間は変更されない
- データは変更されている間は読み込まれない
- データは破棄された後はアクセスされない

これらの制約は**借用チェッカー**によって実施されます。借用チェッカーはRustコンパイラのサブシステムで、Rustコミュニティでジョークやミームの対象になることがよくあります。

所有権はRustの重要な概念であり、この言語をユニークにするものです。
所有権によって、Rustは**パフォーマンスを損なうことなくメモリ安全性**を提供することができます。
Rustについて、これらすべてが同時に真です：

1. 実行時ガベージコレクタはありません
2. 開発者として、メモリを直接管理する必要はほとんどありません
3. ダングリングポインタ、二重解放、その他のメモリ関連バグを引き起こすことはできません

Python、JavaScript、Javaのような言語は2.と3.を提供しますが、1.はありません。  
CやC++のような言語は1.を提供しますが、2.も3.もありません。

あなたの背景によっては、3.は少し不可解に聞こえるかもしれません：「ダングリングポインタ」とは何でしょうか？
「二重解放」とは何でしょうか？なぜそれらは危険なのでしょうか？  
心配しないでください：コースの残りの部分でこれらの概念をより詳しく扱います。

しかし、今のところは、Rustの所有権システムの中で作業する方法を学ぶことに焦点を当てましょう。

## 所有者

Rustでは、各値には**所有者**があり、コンパイル時に静的に決定されます。
任意の時点で各値には一人の所有者しかいません。

## ムーブセマンティクス

所有権は転送することができます。

値を所有している場合、例えば、別の変数に所有権を転送できます：

```rust
let a = "hello, world".to_string(); // <- `a`がStringの所有者です
let b = a;  // <- `b`が今Stringの所有者です
```

Rustの所有権システムは型システムに組み込まれています：各関数は、引数と_どのように_相互作用したいかをシグネチャで宣言しなければなりません。

これまで、すべてのメソッドと関数は引数を**消費**してきました：所有権を取得してきました。
例えば：

```rust
impl Ticket {
    pub fn description(self) -> String {
        self.description
    }
}
```

`Ticket::description`は呼び出された`Ticket`インスタンスの所有権を取得します。  
これは**ムーブセマンティクス**として知られています：値（`self`）の所有権は呼び出し元から被呼び出し元に**移動**され、呼び出し元はもうそれを使用できません。

これは前に見たエラーメッセージでコンパイラが使用した言語とまったく同じです：

```text
error[E0382]: use of moved value: `ticket`
  --> src/main.rs:30:43
   |
25 |     let ticket = Ticket::new(/* */);
   |         ------ move occurs because `ticket` has type `Ticket`, 
   |                which does not implement the `Copy` trait
26 |     if ticket.status() == "To-Do" {
   |               -------- `ticket` moved due to this method call
...
30 |         println!("Your next task is: {}", ticket.title());
   |                                           ^^^^^^ 
   |                                 value used here after move
   |
note: `Ticket::status` takes ownership of the receiver `self`, 
      which moves `ticket`
  --> src/main.rs:12:23
   |
12 |         pub fn status(self) -> String {
   |                       ^^^^
```

特に、これは`ticket.status()`を呼び出すときに展開される一連のイベントです：

- `Ticket::status`は`Ticket`インスタンスの所有権を取得します
- `Ticket::status`は`self`から`status`を抽出し、`status`の所有権を呼び出し元に転送し戻します
- `Ticket`インスタンスの残りは破棄されます（`title`と`description`）

`ticket.title()`を介して`ticket`を再び使用しようとすると、コンパイラは文句を言います：`ticket`の値はもう存在せず、もう所有していないので、もう使用できません。

_有用な_アクセサメソッドを構築するには、**参照**を使い始める必要があります。

## 借用

所有権を取得せずに変数の値を読み込むことができるメソッドを持つことは望ましいことです。  
そうでなければプログラミングは非常に制限されるでしょう。Rustでは、これは**借用**を介して行われます。

値を借用するたびに、それへの**参照**を取得します。  
参照にはそれらの権限がタグ付けされています[^refine]：

- 不変参照（`&`）は値を読み込むことを許可しますが、変更することは許可しません
- 可変参照（`&mut`）は値を読み込み、変更することを許可します

Rustの所有権システムの目標に戻ります：

- データは読み込まれている間は変更されない
- データは変更されている間は読み込まれない

これら2つのプロパティを保証するために、Rustは参照にいくつかの制限を導入しなければなりません：

- 同じ値に対して可変参照と不変参照を同時に持つことはできません
- 同じ値に対して複数の可変参照を同時に持つことはできません
- 所有者は借用されている間は値を変更できません
- 可変参照がない限り、必要なだけ多くの不変参照を持つことができます

ある意味では、不変参照を値の「読み取り専用」ロックとして考えることができ、可変参照は「読み書き」ロックのようなものです。

これらの制限はすべて借用チェッカーによってコンパイル時に実施されます。

### 構文

実際に値を借用するにはどうすればよいでしょうか？  
**変数の前に**`&`または`&mut`を追加することで、その値を借用します。
ただし注意してください！**型の前の**同じシンボル（`&`と`&mut`）は異なる意味を持ちます：
それらは異なる型、元の型への参照を表します。

例えば：

```rust
struct Configuration {
    version: u32,
    active: bool,
}

fn main() {
    let config = Configuration {
        version: 1,
        active: true,
    };
    // `b`は`config`の`version`フィールドへの参照です。
    // `b`の型は`&u32`です。これは
    // `u32`値への参照を含んでいるからです。
    // `&`演算子を使用して`config.version`を借用することで
    // 参照を作成します。
    // 同じシンボル（`&`）、コンテキストによって異なる意味！
    let b: &u32 = &config.version;
    //     ^ 型注釈は必要ありません、 
    //       何が起きているかを明確にするためだけにあります
}
```

同じ概念が関数の引数と戻り値の型にも適用されます：

```rust
// `f`は`u32`への可変参照を引数として取り、
// `number`という名前にバインドされます
fn f(number: &mut u32) -> &u32 {
    // [...]
}
```

## 息を吸って、息を吐いて

Rustの所有権システムは最初は少し圧倒的かもしれません。  
しかし心配しないでください：練習で自然になります。
この章の残りとコースの残りを通して、たくさんの練習をすることになります！
それらに慣れ親しみ、どのように動作するかを本当に理解できるよう、各概念を複数回再訪します。

この章の終わりに向けて、なぜRustの所有権システムがこのように設計されているのかを説明します。
当面は、_どのように_を理解することに焦点を当ててください。各コンパイラエラーを学習の機会として捉えてください！

[^refine]: これは始めるのに素晴らしいメンタルモデルですが、_完全な_全体像を捉えてはいません。
参照の理解を[コースの後半で](../07_threads/06_interior_mutability.md)洗練します。