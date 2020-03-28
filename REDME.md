マーベリック開発部門トレーニング用のレポジトリです。社内の機密事項は含まないので、公開しています。

マーベリック開発部門のメンバーはこちらから[「課題提出トレーニングのポリシー」](https://docs.google.com/document/d/18SWcDVK_urhA4FOpwyjax2d6ZhDA-k9L4mFQnQrMhec/edit#)も確認して下さい。

「[ScalikeJDBC公式ドキュメント](http://scalikejdbc.org/)」から課題を出します。 この課題をこなすことがScalikeJDBCトレーニングのゴールです。
GitHubにPull Requestとして提出する課題を用意し、課題の提出によって達成感を味わってもらうようにします。

提出する際にはこのレポジトリを個人のレポジトリとしてForkして下さい。
課題一つ一つの提出をPull Requestとして挙げて下さい。Pull RequestのApprovalをもって課題完了の確認とします。

## この課題で身につく能力

「〇〇の内容をマスターする」だと際限なく課題の量が膨れ上がるので、課題は必要な量だけにします。書籍なら2〜3割内容を把握できる分量が目安です。ソフトウェア開発技術の勉強は一生つづくので、技術者はこの課題だけでなく、結局様々な分野を様々な方法で勉強をするからです。

- 手早くScalikeJDBCアプリケーションのBareboneを作ってデータベースと疎通確認できる
- SQLコネクションの概念を説明できる
- sql”"を実行してデータベースからSELECT、UPDATE、INSERTできる
- トランザクションで2テーブル間のデータ不整合を防ぐサンプルコードが書ける
- QueryDSLとsql”"の違いを把握している、Query DSLは使いこなせなくてよい

## 参考書籍・オンラインテキスト:

- [Scalike JDBC公式ドキュメント](http://scalikejdbc.org/)
- [ScalikeJDBC / Skinny ORM Beginners' Guide](https://speakerdeck.com/seratch/skinny-orm-beginners-guide-1)
  - ScalikeJDBC作成者の瀬良さんご本人によるスライド
- [段階的に理解する O/R マッピング](https://qiita.com/ts7i/items/c23e50b5ee29887c446c)
  - ScalikeJDBC自体に関する話ではありませんが、O/Rマッパーの一般に関する話です
  - ScalikeJDBC作成者の瀬良さんもツイートしています
    - https://twitter.com/seratch_ja/status/1208970929889349632
    - > これ、良記事だと思います。私が Scala の OSS でやったのは ScalikeJDBC をレベル 1 から始めてレベル 4 まで行き着き、レベル 5 のために Skinny ORM をつくったという流れだったので、思考をトレースされる感ありました。
    - > ScalikeJDBC はレベル 1 からと書いたけど、レベル 2 からスタートでしたね。良い記事なのにあまり注目されてなかったけど、この tweet も多少役に立ったのか、結構読まれているようなのでよかった。
- [Java ORマッパー選定のポイント #jsug](https://www.slideshare.net/masatoshitada7/java-or-jsug)
  - これもScalikeJDBC自体に関する話ではありませんが、O/Rマッパーの一般に関する話です
  