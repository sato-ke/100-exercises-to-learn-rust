# チケットのモデリング、パート2

前の章で作成した`Ticket`構造体は良い出発点ですが、まだ「初心者のRustaceanです！」と叫んでいるようなものです。

この章では、Rustのドメインモデリングスキルを磨いていきます。途中でいくつかの新しい概念を導入する必要があります：

- `enum`、データモデリングにおけるRustの最も強力な機能の一つ
- `Option`型、null許容値をモデル化するため
- `Result`型、回復可能なエラーをモデル化するため
- `Debug`と`Display`トレイト、出力のため
- `Error`トレイト、エラー型をマークするため
- `TryFrom`と`TryInto`トレイト、失敗する可能性のある変換のため
- Rustのパッケージシステム、ライブラリとは何か、バイナリとは何か、サードパーティクレートの使用方法の説明