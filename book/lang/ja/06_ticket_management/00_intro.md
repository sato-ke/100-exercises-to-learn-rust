# イントロダクション

前の章では、`Ticket`を単体で定義しました。フィールドやその制約を定義し、Rustでの最適な表現方法を学びましたが、`Ticket`がより大きなシステムにどのように組み込まれるかは考慮していませんでした。
この章では、`Ticket`を中心とした簡単なワークフローを構築し、チケットを保存・取得するための（基本的な）管理システムを導入します。

このタスクを通じて、以下のような新しいRustの概念を探求する機会が得られます：

- スタック割り当て配列
- `Vec`、成長可能な配列型
- `Iterator`と`IntoIterator`、コレクションの反復処理のために
- スライス（`&[T]`）、コレクションの一部を扱うために
- ライフタイム、参照がどのくらい有効かを記述するために
- `HashMap`と`BTreeMap`、2つのキー値データ構造
- `Eq`と`Hash`、`HashMap`のキーを比較するために
- `Ord`と`PartialOrd`、`BTreeMap`で使用するために
- `Index`と`IndexMut`、コレクション内の要素にアクセスするために