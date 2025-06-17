# ようこそ

**「Rustを学ぶ100の演習」**へようこそ！

このコースでは、Rustの中核概念を1つずつ演習を通して学びます。\
Rustの構文、型システム、標準ライブラリ、そしてエコシステムについて学習します。

私たちはRustの事前知識を前提としませんが、少なくとも他のプログラミング言語を
1つは知っていることを前提としています。
また、システムプログラミングやメモリ管理の事前知識も前提としません。これらの
トピックはコース内でカバーされます。

つまり、ゼロから始めます！\
小さく管理しやすいステップでRustの知識を構築していきます。
コースの終わりまでに、約100個の演習を解き終え、小規模から中規模のRustプロジェクトで
快適に作業できるようになるでしょう。

## 方法論

このコースは「実践による学習」の原則に基づいています。\
インタラクティブで実践的になるよう設計されています。

[Mainmatter](https://mainmatter.com/rust-consulting/)は、このコースを
4日間の教室環境で提供するために開発しました：各参加者は自分のペースで
レッスンを進め、経験豊富なインストラクターがガイダンスを提供し、
質問に答え、必要に応じてトピックを深掘りします。\
次の指導付きセッションには[私たちのウェブサイト](https://ti.to/mainmatter/rust-from-scratch-jan-2025)から登録できます。
企業向けのプライベートセッションを開催したい場合は、[お問い合わせください](https://mainmatter.com/contact/)。

独学でコースを進めることもできますが、行き詰まった時に助けてくれる友人や
メンターを見つけることをお勧めします。すべての演習の解答は
[GitHubリポジトリの`solutions`ブランチ](https://github.com/mainmatter/100-exercises-to-learn-rust/tree/solutions)
で見つけることができます。

## フォーマット

コース教材は[ブラウザで閲覧](https://rust-exercises.com/100-exercises/)することも、オフライン読書用に[PDFファイルとしてダウンロード](https://rust-exercises.com/100-exercises-to-learn-rust.pdf)することもできます。\
印刷版を希望する場合は、[Amazonでペーパーバック版を購入](https://www.amazon.com/dp/B0DJ14KQQG/)できます。

## 構造

画面の左側に、コースがセクションに分かれているのが見えます。
各セクションでは、Rust言語の新しい概念や機能を紹介します。\
理解を確認するために、各セクションには解決すべき演習が組み合わされています。

演習は[付属のGitHubリポジトリ](https://github.com/mainmatter/100-exercises-to-learn-rust)
で見つけることができます。\
コースを始める前に、リポジトリをローカルマシンにクローンしてください：

```bash
# GitHubでSSHキーを設定している場合
git clone git@github.com:mainmatter/100-exercises-to-learn-rust.git
# それ以外の場合は、HTTPSのURLを使用してください：
#   https://github.com/mainmatter/100-exercises-to-learn-rust.git
```

また、ブランチで作業することをお勧めします。これにより、進捗を簡単に追跡でき、
必要に応じてメインリポジトリから更新を取り込むことができます：

```bash
cd 100-exercises-to-learn-rust
git checkout -b my-solutions
```

すべての演習は`exercises`フォルダにあります。
各演習はRustパッケージとして構成されています。
パッケージには、演習自体、何をすべきかの指示（`src/lib.rs`内）、そして
解答を自動的に検証するためのテストスイートが含まれています。

### ツール

このコースを進めるには、以下が必要です：

- [**Rust**](https://www.rust-lang.org/tools/install)。
  システムに`rustup`がすでにインストールされている場合は、`rustup update`（またはシステムにRustをインストールした方法に応じた適切なコマンド）を実行して、最新の安定版を使用していることを確認してください。
- _（オプションですが推奨）_ Rustの自動補完サポートを備えたIDE。
  以下のいずれかをお勧めします：
  - [RustRover](https://www.jetbrains.com/rust/)
  - [`rust-analyzer`](https://marketplace.visualstudio.com/items?itemName=matklad.rust-analyzer)拡張機能を備えた[Visual Studio Code](https://code.visualstudio.com)

### ワークショップランナー、`wr`

解答を検証するために、コースを進めるためのツールも提供しています：`wr` CLI、「ワークショップランナー」の略です。
`wr`は[そのウェブサイト](https://mainmatter.github.io/rust-workshop-runner/)の指示に従ってインストールしてください。

`wr`をインストールしたら、新しいターミナルを開き、リポジトリのトップレベルフォルダに移動します。
`wr`コマンドを実行してコースを開始します：

```bash
wr
```

`wr`は現在の演習の解答を検証します。\
現在のセクションの演習を解決するまで、次のセクションに進まないでください。

> コースを進めるにつれて、解答をGitにコミットすることをお勧めします。
> これにより、進捗を簡単に追跡でき、必要に応じて既知のポイントから「再起動」できます。

コースをお楽しみください！

## 著者

このコースは[Mainmatter](https://mainmatter.com/rust-consulting/)の
プリンシパルエンジニアリングコンサルタントである[Luca Palmieri](https://www.lpalmieri.com/)によって書かれました。\
LucaはRust 2018から、最初はTrueLayerで、その後AWSでRustを使用しています。\
Lucaは、Rustでバックエンドアプリケーションを構築する方法を学ぶための頼りになるリソースである
["Zero to Production in Rust"](https://zero2prod.com)の著者です。\
彼はまた、[`cargo-chef`](https://github.com/LukeMathWalker/cargo-chef)、
[Pavex](https://pavex.dev)、[`wiremock`](https://github.com/LukeMathWalker/wiremock-rs)
を含む、さまざまなオープンソースRustプロジェクトの著者およびメンテナーでもあります。