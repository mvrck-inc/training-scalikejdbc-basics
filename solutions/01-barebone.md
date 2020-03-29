## この課題で身につく能力

手早くScalikeJDBCアプリケーションのBareboneを作ってデータベースと疎通確認できる

## 課題

以下の課題の結果をこのファイルに貼り付けて、Pull Requestを送る形で提出してください

1. MySQLコマンドラインクライアントから、MySQLサーバーにログインして以下を実行してください

```sql
> mysql -h 127.0.0.1 -u root -p

CREATE DATABASE scalikejdbc_basics_db;

USE scalikejdbc_basics_db;

CREATE TABLE emp (
  id int NOT NULL,
  name VARCHAR(1024) NOT NULL,
  PRIMARY KEY(id) 
) CHARACTER SET utf8mb4; -- MySQL 5.7だと、utf8mb4でないと日本語を正しく扱ってくれません

INSERT INTO emp (id, name) VALUES (123, 'Richard Imaoka');
INSERT INTO emp (id, name) VALUES (456, 'Richard Yamaoka');

SELECT * from emp;
```

解答)

まずはコマンドラインからログイン

```
> mysql -h 127.0.0.1 -u root -p
Enter password: ****

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 32
Server version: 8.0.19 MySQL Community Server - GPL
```

つぎにSQLを実行

```sql
mysql> CREATE DATABASE scalikejdbc_basics_db;

Query OK, 1 row affected (0.02 sec)

mysql> USE scalikejdbc_basics_db;
Database changed

mysql>
mysql> CREATE TABLE emp (
    ->   id int NOT NULL,
    ->   name VARCHAR(1024) NOT NULL,
    ->   PRIMARY KEY(id)
    -> ) CHARACTER SET utf8mb4;
Query OK, 0 rows affected (0.14 sec)

mysql> INSERT INTO emp (id, name) VALUES (123, 'Richard Imaoka');
Query OK, 1 row affected (0.01 sec)

mysql> INSERT INTO emp (id, name) VALUES (456, 'Richard Yamaoka');
Query OK, 1 row affected (0.01 sec)

mysql> SELECT * from emp;
+-----+-----------------+
| id  | name            |
+-----+-----------------+
| 123 | Richard Imaoka  |
| 456 | Richard Yamaoka |
+-----+-----------------+
2 rows in set (0.00 sec)
```

---
2. `sbt new`を使って新規プロジェクトを作成し、`build.sbt`の`libraryDependencies`を以下に置き換えてください

https://github.com/mvrck-inc/training-scala-basics/blob/master/solutions/01-dev-env.md

```scala
libraryDependencies ++= Seq (
  "org.scalikejdbc" %% "scalikejdbc" % "3.4.1",
  "mysql" % "mysql-connector-java" % "8.0.19", //MySQL 5.7 Serverと接続できるらしい https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-versions.html
  "ch.qos.logback"  %  "logback-classic" % "1.2.3"
)
```

解答) sbt newコマンドでIDEで開くことのできるScalaプロジェクトの雛形をつくります。

```
sbt new scala/scala-seed.g8
//この後ターミナル/コンソールでいくつかの質問に対する答えを入力する
```

出来上がるプロジェクト雛形の構成はこのような感じです。

```
  +-project
  | +-build.properties
  |
  +-src
  | +-main
  |   +-scala
  |     +-example
  |       +-Main.scala //Example.scalaをMain.scalaに名前変更。これは好みの問題
  +-test
  +-build.sbt
```

上記のbuild.sbtの`libraryDependencies`を書き換えれば完了です。build.sbt全体はこのようになります。

```scala
import Dependencies._

ThisBuild / scalaVersion     := "2.13.1"
ThisBuild / version          := "0.1.0-SNAPSHOT"
ThisBuild / organization     := "com.example"
ThisBuild / organizationName := "example"

lazy val root = (project in file("."))
  .settings(
    name := "training-scalikejdbc-basics",
    libraryDependencies ++= Seq (
      "org.scalikejdbc" %% "scalikejdbc" % "3.4.1",
      "mysql" % "mysql-connector-java" % "8.0.19",
      "ch.qos.logback"  %  "logback-classic" % "1.2.3"
    )
  )
```

---
3. src/main/resources/application.confを作成し、以下の内容を貼り付けてください

```
# MySQL example
db.default.driver="com.mysql.jdbc.Driver"
db.default.url="jdbc:mysql://127.0.0.1:3306/scalikejdbc_basics_db"
db.default.user="root"
db.default.password="root"
```

解答) ただファイルを作って貼り付けるだけです。プロジェクトのディレクトリ構成はこのようになります。

```
  +-project
  | +-build.properties
  |
  +-src
  | +-main
  |   +-resources
  |     +-application.conf //<- ココ!!!
  |   +-scala
  |     +-example
  |       +-Main.scala 
  +-test
  +-build.sbt
```

ちなみに、Windows機をつかってこのようなエラーが出ることがあります。

> Caused by: com.mysql.cj.exceptions.InvalidConnectionAttributeException: The server time zone value '**************' is unrecognized or represents more than one time zone. You must configure either the server or JDBC driver (via the 'serverTimezone' configuration property) to use a more specifc time zone value if you want to utilize time zone support.

その場合は以下のように`?serverTimezone=UTC`をJDBC URLに加えてやると解決するでしょう。

```
# Windows機ならserverTimeZone設定が必要かもしれない
# db.default.url="jdbc:mysql://127.0.0.1:3306/scalikejdbc_basics_db?serverTimezone=UTC"
```

実際にはこの解決方法、クライアント側からコネクション設立時にServer Time Zoneを上書きするという、好ましくない解決策です。しかし、ローカル開発環境なら問題ではありません。

https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-reference-configuration-properties.html
> serverTimezone
> Override detection/mapping of time zone. Used when time zone from server doesn't map to Java time zone

なぜ好ましくないかといと、Server Time Zoneは当然サーバー側のタイムゾーン設定なので、当然サーバー側の設定で管理したいからです。

https://dev.mysql.com/doc/refman/8.0/en/time-zone-support.html

> The server current time zone. The global time_zone system variable indicates the time zone the server currently is operating in. The initial time_zone value is 'SYSTEM', which indicates that the server time zone is the same as the system time zone.

ところが`SET GLOBAL time_zone = UTC;`をやろうとしてもエラーが出て、タイムゾーンの値が不正です!!みたいなエラーが出ます。これはどうもタイムゾーンテーブルのロードをしないと解決しないようで、以下で解決されるかなりめんどくさい手順を踏まなくてはなりません。

https://dev.mysql.com/doc/refman/8.0/en/time-zone-support.html
"Populating the Time Zone Tables"

というわけで、この課題をやるためだけなら諦めて`serverTimezone=UTC`をJDBC URLに加えるのが妥当だと思います。

---
4. 以下をMain.scalaに貼り付けて`sbt run`で実行してください

```scala
package example

import scalikejdbc._

object Main {
  def main(args: Array[String]): Unit = {
    val id = 123

    val name: Option[String] = DB readOnly { implicit session =>
      sql"select name from emp where id = ${id}".map(rs => rs.string("name")).single.apply()
    }

    println(name)
  }
}
```

解答) 

こんな感じで出力が見えます。途中Stack Traceが見えてますがこれはエラーではないです。ScalikeJDBCにはSQL実行のたびにStack Traceをログ出力機能があります。
エラーではない証拠に、出力の最後は`Some(Richard Imaoka)`と`println(name)`で期待される結果が出力されています。

```
>sbt run
[info] running example.Main
Loading class `com.mysql.jdbc.Driver'. This is deprecated. The new driver class is `com.mysql.cj.jdbc.Driver'. The driver is automatically registered via the SPI and manual loading of the driver class is generally unnecessary.
14:20:37.196 [run-main-0] DEBUG scalikejdbc.ConnectionPool$ - Registered connection pool : ConnectionPool(url:jdbc:mysql://127.0.0.1:3306/scalikejdbc_basics_db?serverTimezone=UTC, user:root) using factory : <default>
14:20:37.671 [run-main-0] DEBUG scalikejdbc.StatementExecutor$$anon$1 - SQL execution completed

  [SQL Execution]
   select name from emp where id = 123; (26 ms)

  [Stack Trace]
    ...
    example.Main$.$anonfun$main$1(Main.scala:12)
    scalikejdbc.DBConnection.readOnly(DBConnection.scala:211)
    scalikejdbc.DBConnection.readOnly$(DBConnection.scala:209)
    scalikejdbc.DB.readOnly(DB.scala:60)
    scalikejdbc.DB$.$anonfun$readOnly$1(DB.scala:174)
    scalikejdbc.LoanPattern.using(LoanPattern.scala:22)
    scalikejdbc.LoanPattern.using$(LoanPattern.scala:20)
    scalikejdbc.DB$.using(DB.scala:139)
    scalikejdbc.DB$.readOnly(DB.scala:173)
    example.Main$.main(Main.scala:11)
    example.Main.main(Main.scala)
    java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
    java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    java.base/java.lang.reflect.Method.invoke(Method.java:566)
    ...

Some(Richard Imaoka)
[success] Total time: 1 s, completed Mar 29, 2020, 2:20:37 PM
```