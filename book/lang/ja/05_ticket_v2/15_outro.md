# まとめ

ドメインモデリングに関しては、細部に悪魔が宿ります。  
Rustは、ドメインの制約を型システムで直接表現するための幅広いツールを提供しますが、それを正しく行い、慣用的に見えるコードを書くには練習が必要です。

`Ticket`モデルの最終的な改良で章を閉じましょう。  
それぞれの制約をカプセル化するために、`Ticket`の各フィールドに新しい型を導入します。  
誰かが`Ticket`フィールドにアクセスするたびに、有効であることが保証された値（`String`ではなく`TicketTitle`）が返されます。コードの他の場所でタイトルが空であることを心配する必要がありません：  
`TicketTitle`を持っている限り、それは**構築によって**有効であることがわかります。

これは、Rustの型システムを使用してコードをより安全で表現力豊かにする方法の一例にすぎません。

## 参考文献

- [Parse, don't validate](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/)
- [Using types to guarantee domain invariants](https://www.lpalmieri.com/posts/2020-12-11-zero-to-production-6-domain-modelling/)