## この課題で身につく能力

sql”"を実行してデータベースからSELECT、UPDATE、INSERTできる

### ねらい

公式ドキュメントの[Operationsページ](http://scalikejdbc.org/documentation/operations.html)から一部抜き出して出題します。
クエリの書き方、実行の仕方に慣れてください。

## 課題

以下の課題の結果をこのファイルに貼り付けて、Pull Requestを送る形で提出してください

1. Scalaプロジェクトを作成しIDEから開いた後、以下のコードを含むmainメソッドを定義して、実行結果を貼り付けて提出してください

```scala
import scalikejdbc._
val id = 123
// simple example
val name: Option[String] = DB readOnly { implicit session =>
  sql"select name from emp where id = ${id}".map(rs => rs.string("name")).single.apply()
}
println(name)
```

なお、データは[01-barebone](./01-barebone.md)で用意したのと同じ方法で用意してください。

```sql
> mysql -h 127.0.0.1 -u root -p

-- これでテーブルの中身が残っていればデータの準備はOKです。
USE scalikejdbc_basics_db;
select * from emp;
```

テーブルの中身が残っていなければ、再度同じ手順で用意してください。

```sql
CREATE DATABASE scalikejdbc_basics_db;

USE scalikejdbc_basics_db;

CREATE TABLE emp (
  id int NOT NULL,
  name VARCHAR(1024) NOT NULL,
  PRIMARY KEY(id) 
) CHARACTER SET utf8mb4; -- MySQL 5.7だと、utf8mb4でないと日本語を正しく扱ってくれません

INSERT INTO emp (id, name) VALUES (123, 'Richard Imaoka');
INSERT INTO emp (id, name) VALUES (456, 'Richard Yamaoka');
```

解答) [01-barebone](./01-barebone.md)で実行したものと同じです。

```scala
package example

import scalikejdbc._
import scalikejdbc.config._

object Main {
  def main(args: Array[String]): Unit = {
    DBs.setupAll()

    val id = 123
    val name: Option[String] = DB readOnly { implicit session =>
      sql"select name from emp where id = ${id}".map(rs => rs.string("name")).single.apply()
    }
    println(name)
  }
}
```

注目してほしい点は:
 - `DB readOnly`メソッドの中に`sql`を含むクエリが書いてある
 - `implicit session =>`は`DB readOnly`呼び出しに毎回必要なおまじないみたいなものだと思ってください。解説が必要なら解説しますが、難しいですよ。
 - `sql"..."`は`def sql`メソッドとして実装されている
    - https://github.com/scalikejdbc/scalikejdbc/blob/3.4.1/scalikejdbc-core/src/main/scala/scalikejdbc/SQLInterpolationString.scala#L14
 - `map`メソッドでSQL分の実行結果をStringに変換している
 - `where`句を使っていたとしても、原理的には複数のレコードが返ってくる可能性があるため、`single`メソッドで単一のレコードを扱うことを明示
 - 戻り値が`Option`になっている。これは単一のレコードを扱うといっても、レコードがゼロ件の可能性もあるため。
 

実行結果はこちら。

```
>sbt run
Some(Richard Imaoka)
[success] Total time: 1 s, completed Mar 29, 2020, 5:36:45 PM
```

---
2. 以下のコードを含むmainメソッドを定義して、実行結果を貼り付けて提出してください

```scala
// defined mapper as a function
val nameOnly = (rs: WrappedResultSet) => rs.string("name")
val name: Option[String] = DB readOnly { implicit session =>
  sql"select name from emp where id = ${id}".map(nameOnly).single.apply()
}
println(name)
```

解答) ScalikeJDBCではSQLの結果は`WrappedResultSet`で返されます。一方この`WrappedResultSet`は通常Scalaアプリケーションの中ではそのまま使うわけではなく、
適宜自分で定義した型のインスタンスに変換して使います。自作のcase classなどを使うことが多いでしょう。(このページの内の次の課題3. がcase classを使った例です。)

今回は自作のクラスでなくStringにマッピングします。`WrappedResultSet => String`。それが次の部分です。

```
val nameOnly = (rs: WrappedResultSet) => rs.string("name")
```

```scala
package example

import scalikejdbc._
import scalikejdbc.config._

object Main {
  def main(args: Array[String]): Unit = {
    DBs.setupAll()

    val id = 123
    // defined mapper as a function
    val nameOnly = (rs: WrappedResultSet) => rs.string("name")
    val name: Option[String] = DB readOnly { implicit session =>
      sql"select name from emp where id = ${id}".map(nameOnly).single.apply()
    }
    println(name)
  }
}
```

実行結果はこちら。

```
>sbt run
Some(Richard Imaoka)
[success] Total time: 16 s, completed Mar 29, 2020, 5:38:22 PM
```

---
3. 以下のコードを含むmainメソッドを定義して、実行結果を貼り付けて提出してください

```scala
import scalikejdbc._
val id = 123

// define a class to map the result
case class Emp(id: String, name: String)
val emp: Option[Emp] = DB readOnly { implicit session =>
  sql"select id, name from emp where id = ${id}"
    .map(rs => Emp(rs.string("id"), rs.string("name"))).single.apply()
}
```

解答) 直前の課題2. では`WrappedResultSet => String`というマッピングでしたが、今回は`WrappedResultSet => Emp`です。自作の`case class Emp`を使っています。

```scala
package example

import scalikejdbc._
import scalikejdbc.config._

object Main {
  def main(args: Array[String]): Unit = {
    DBs.setupAll()

    val id = 123
    // define a class to map the result
    case class Emp(id: String, name: String)
    val emp: Option[Emp] = DB readOnly { implicit session =>
      sql"select id, name from emp where id = ${id}"
        .map(rs => Emp(rs.string("id"), rs.string("name"))).single.apply()
    }
    println(emp)
  }
}
```

```
>sbt run
Some(Emp(123,Richard Imaoka))
[success] Total time: 4 s, completed Mar 29, 2020, 6:03:45 PM
```

---
4. 以下のコードを含むmainメソッドを定義して、実行結果を貼り付けて提出してください

```scala
import scalikejdbc._

case class Emp(id: String, name: String)

val id2 = 789
val name2 = "Richard Yoshioka"
DB localTx { implicit session =>
  sql"insert into emp (id, name) values (${id2}, ${name2})".update.apply()
}

val emp2: Option[Emp] = DB readOnly { implicit session =>
  sql"select id, name from emp where id = ${id2}".map(rs => Emp(rs.string("id"), rs.string("name"))).single.apply()
}
println(emp2)
```

解答) 実行結果はこちら。今回はINSERT文なので`DB readOnly`ではなく`DB localTx`です。

```
> sbt run
Some(Emp(789,Richard Yoshioka))
[success] Total time: 4 s, completed Mar 29, 2020, 9:45:28 PM
```

---
5. 以下のコードを含むmainメソッドを定義して、実行結果を貼り付けて提出してください

```scala
import scalikejdbc._

case class Emp(id: String, name: String)

val id2 = 789
val newName2 = "Riho Yoshioka"
DB localTx { implicit session =>
  sql"update emp set name = ${newName2} where id = ${id2}".update.apply()
}
val newEmp2: Option[Emp] = DB readOnly { implicit session =>
  sql"select id, name from emp where id = ${id2}".map(rs => Emp(rs.string("id"), rs.string("name"))).single.apply()
}
println(newEmp2)
```

解答) 実行結果はこちら。今回はUPDATE文です。

```
> sbt run
Some(Emp(789,Riho Yoshioka))
[success] Total time: 4 s, completed Mar 29, 2020, 9:53:22 PM
```

---
6. 以下のコードを含むmainメソッドを定義して、実行結果を貼り付けて提出してください

```scala
import scalikejdbc._

case class Emp(id: String, name: String)

val id2 = 789
DB localTx { implicit session =>
  sql"delete from emp where id = ${id2}".update.apply()
}
val newEmp2: Option[Emp] = DB readOnly { implicit session =>
  sql"select id, name from emp where id = ${id2}".map(rs => Emp(rs.string("id"), rs.string("name"))).single.apply()
}
println(newEmp2)
```

解答) 実行結果はこちら。今回はDELETE文なので、DELETE後のSELECTではレコードが見つからず、Noneが返されます。

```
> sbt run
None
[success] Total time: 4 s, completed Mar 29, 2020, 10:07:34 PM
```