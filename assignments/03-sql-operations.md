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

2. 以下のコードを含むmainメソッドを定義して、実行結果を貼り付けて提出してください

```scala
// defined mapper as a function
val nameOnly = (rs: WrappedResultSet) => rs.string("name")
val name: Option[String] = DB readOnly { implicit session =>
  sql"select name from emp where id = ${id}".map(nameOnly).single.apply()
}
println(name)
```

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
