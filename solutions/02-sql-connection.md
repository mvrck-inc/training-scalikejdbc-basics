## この課題で身につく能力

SQLコネクションの概念を説明できる

## 課題

以下の課題は口頭試問です。メモをこのファイルに貼り付けて、Pull Requestを送る形で提出してください。

1. データベース・コネクションを以下の用語を使って、図を描きながら說明してください。

- サーバー
- クライアント
- コネクション
- SQL文
- セキュリティ
- ユーザー名、パスワード
- セッション

解答) まずデータベースにはサーバーとクライアントがあります。クライアントはSQL文を発行して、サーバーがそれを受け付けて処理結果をクライアントに返す、というのが大まかな流れです。
Mac OSでhomebrewを使ってMySQLをインストールした場合、クライアントには`mysql`コマンドが、サーバーには`mysql.Server`コマンドが割り当てられます。

クライアントがSQL文をサーバーに送る前にはコネクションを確立しなくてはなりません。

コネクションを確立したクライアントのみ、サーバーにSQL文を送ることが出来ますが、誰でも簡単にコネクションを確立できてはセキュリティを保てません。
そのため、リレーショナル・データベースは基本的にコネクション確立時に、ユーザー名とパスワードがあっているか検査します。

コネクションが確立すると、セッション情報というものが保持されます。セッション情報はコネクションひとつひとつに固有な情報で、
その中にはタイムゾーンや文字コードなどが含まれます。ただし厄介なのが、例えば文字コードはセッション毎の設定の他にテーブルごとの設定やカラムごとの設定などいろいろな場所で設定でき、どこで設定したか、それらの優先順位はどうなっているか、ちゃんと把握しないと予期せぬ動作を引き起こします。

ちなみにユーザー名とパスワードによる検査は、通常セキュリティの世界においては最もセキュアな方法ではありません。
なぜならパスワードは漏洩するからです。繰り返します、パスワードは漏洩します。もう一度繰り返します。パスワードは漏洩します。

ではなぜデータベースの世界ではコネクション確立時に、ユーザー名とパスワードでチェックするという方法が一般的なのかというと、データベースは限られたクライアントのみ接続できる場所に隔離されているからです。
インターネット越しに世界中からアクセスできるわけではなく、ファイアウォールを使って適切にアクセスできるクライアントが限定されています。

---
2. JDBCを以下の単語を使って、図を描きながら用明してください。

- インターフェース
- 実装
- mysql-connector-j (mysql-connector-java 8.0.19)
- JDBC URL

解答) JDBCはJavaが定義している、リレーショナル・データベース接続のためのインターフェースです。インターフェースであるため、それだけでは接続できません。実装が必要です。
mysql-connector-jは、クライアントであるJavaアプリケーションから、MySQLサーバーに接続するための実装です。
JDBCは接続時に必要な情報をJDBC URLとして表現することが一般的です。しかし、JDBC URLで接続に必要な全ての情報を表すわけではなく、例えばユーザ名やパスワードは別途URLに含まずに設定します。


---
3. データベース・コネクション・プールを以下の用語を使って、図を描きながら說明してください。

- データベース・コネクションのコスト
- マルチスレッド
- トランザクション
- loan pattern

解答) こちらの解説を見ると良いでしょう。
https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-usagenotes-j2ee-concepts-connection-pooling.html

> How Connection Pooling Works
> 
> Most applications only need a thread to have access to a JDBC connection when they are actively processing a transaction, which often takes only milliseconds to complete. When not processing a transaction, the connection sits idle. Connection pooling enables the idle connection to be used by some other thread to do useful work.
> 
> In practice, when a thread needs to do work against a MySQL or other database with JDBC, it requests a connection from the pool. When the thread is finished using the connection, it returns it to the pool, so that it can be used by any other threads.
> 
> When the connection is loaned out from the pool, it is used exclusively by the thread that requested it. From a programming point of view, it is the same as if your thread called DriverManager.getConnection() every time it needed a JDBC connection. With connection pooling, your thread may end up using either a new connection or an already-existing connection.

コネクション・プールはひとつの(Java/Scala)アプリケーション内で複数のデータベース・コネクションを維持する必要があるときに使います。
維持、というのがポイントです。というのも、データベース・コネクション確立には時間がかかってしまうので、SQL文発行のたびにクライアントとサーバー間でコネクションを確立しては、データベース問い合わせに時間がかかって仕方がありません。

ココから下は余談で、かなり難しい話です。今はまだわからなくてよいです。ScalikeJDBCの課題全てが終わってから再度よめばわかるかもしれません。
assignmentの01で[ScalikeJDBC公式ドキュメント](http://scalikejdbc.org/documentation/operations.html)から引用した以下のコードについて解説します。


```
DB readOnly { implicit session =>
  sql"select name from emp where id = ${id}".map(rs => rs.string("name")).single.apply()
}
```    

上記の`readOnly`メソッドは以下の実装になっています。
https://github.com/scalikejdbc/scalikejdbc/blob/3.4.1/scalikejdbc-core/src/main/scala/scalikejdbc/DB.scala#L171

```scala
def readOnly[A](execution: DBSession => A)(implicit context: CPContext = NoCPContext, settings: SettingsProvider = SettingsProvider.default): A = {
  val cp = connectionPool(context)
  using(cp.borrow()) { conn =>
    DB(conn, cp.connectionAttributes, settings).autoClose(false).readOnly(execution)
  }
}
```  

上記で`cp`はコネクション・プールであり`cp.borrow()`によってプールからコネクションをひとつ借りています。
`using`は以下で定義されるローン・パターンの実装です。

```scala
trait LoanPattern {
  def using[R <: Closable, A](resource: R)(f: R => A): A = {
    try {
      f(resource)
    } finally {
      try {
        resource.close()
      } catch {
        case NonFatal(e) =>
          loanPatternLogger.warn(s"Failed to close a resource (resource: ${resource.getClass().getName()} error: ${e.getMessage})")
      }
    }
  }
}  
```

教科書どおりのようなきれいなローン・パターンの実装ですね。`resource.close()`はコネクション・プールのcloseメソッドを呼び出していて、
実際にコネクションを完全にクローズするのではなく、プールから借りたコネクションをプールに返却しているだけです。

こうすることでmysql-connector-jのドキュメントでも解説されている以下のメリットが実現されています。
https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-usagenotes-j2ee-concepts-connection-pooling.html

> Benefits of Connection Pooling
>
> Simplified programming model.
>
> When using connection pooling, each individual thread can act as though it has created its own JDBC connection, allowing you to use straightforward JDBC programming techniques.