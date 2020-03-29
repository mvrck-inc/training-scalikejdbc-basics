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

2. `sbt new`を使って新規プロジェクトを作成し、`build.sbt`の`libraryDependencies`を以下に置き換えてください

https://github.com/mvrck-inc/training-scala-basics/blob/master/solutions/01-dev-env.md

```scala
libraryDependencies ++= Seq (
  "org.scalikejdbc" %% "scalikejdbc" % "3.4.1",
  "mysql" % "mysql-connector-java" % "8.0.19", //MySQL 5.7 Serverと接続できるらしい https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-versions.html
  "ch.qos.logback"  %  "logback-classic" % "1.2.3"
)
```

3. src/main/resources/application.confを作成し、以下の内容を貼り付けてください

```
# MySQL example
db.default.driver="com.mysql.jdbc.Driver"
db.default.url="jdbc:mysql://127.0.0.1:3306/scalikejdbc_basics_db"
db.default.user="root"
db.default.password="root"
```

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