## この課題で身につく能力

トランザクションで2テーブル間のデータ不整合を防ぐサンプルコードが書ける

### ねらい

データベースのトランザクションを正しく使って、不整合なデータを防ぐのは本番環境で動くアプリケーションを作る上において非常に重要なスキルです。出来ないとこういう目に会います。
- [２０３室のホテルなのに「予約」数千件　嵐公演で殺到か](https://www.asahi.com/articles/ASH533JQ4H53UNHB004.html)
- [嵐のコンサートがあるとダブルブッキングしてしまうホテル予約システムを作ってみた](https://blog.tokumaru.org/2015/05/blog-post.html)


データベースのトランザクションを「正しく」使うのは実は非常に高度なスキルで、データベースのテーブル設計やパフォーマンス分析、さらに非同期処理について詳しくないと正しく使うことが出来ません。そこでこの課題では「正しく」使うことを意識する前に、まずはScalikeJDBCでトランザクションを用いた処理を書けることを目指します。

- [YouTube - Database Transactions, part 2: Examples, Barry Brown](https://www.youtube.com/watch?v=PguCDI_fi3U)

## 課題

以下の課題の結果をこのファイルに貼り付けて、Pull Requestを送る形で提出してください

1. MySQLにログインして以下のSQLでテーブルを作成してください。

```sql
CREATE TABLE `rooms` (
  `id` int NOT NULL,
  `available` BOOLEAN NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8mb4;


CREATE TABLE `reservations` (
  `id` int NOT NULL AUTO_INCREMENT,
  `room_id` int NOT NULL,
  `customer_id` int NOT NULL,
  PRIMARY KEY (`id`),
  FOREIGN KEY(`room_id`) REFERENCES rooms(id)
) ENGINE=INNODB DEFAULT CHARSET=utf8mb4 AUTO_INCREMENT=1;
```

解答) テーブルを作るだけなので省略。提出の際は必ずテーブル作成の結果を貼り付けてください。

---
2. `sbt new`からScalaプロジェクトを作成して(課題[03-sql-operations.md](./03-sql-operations.md)のプロジェクトを再利用すると良いでしょう)、DataInsert.scalaファイルを作成し、
mainメソッドの中でroomsテーブルの中に10件のレコードを挿入してください

解答) 

```scala
package example

import scalikejdbc._
import scalikejdbc.config._

object DataInsert {
  def main(args: Array[String]): Unit = {
    //ログ出力が大量になるので割愛
    GlobalSettings.loggingSQLAndTime = LoggingSQLAndTimeSettings(
      enabled = false
    )
    
    DBs.setupAll()

    case class Room(id: Int, available: Boolean)

    for (roomId <- 1 to 10) {
      DB localTx { implicit session =>
        val roomOpt = sql"SELECT id, available FROM rooms WHERE id = ${roomId}".map {rs => Room(rs.int("id"), rs.boolean("available"))}.single().apply()
        val result = roomOpt match {
          case None => sql"INSERT INTO rooms (id, available) VALUES (${roomId}, true)".update().apply()
          case Some(_) => 1//Do nothing. It's already there
        }
        if (result != 1) throw new Exception(s"Failed to insert room id = ${roomId}")
      }
    }
  }
}
```

`sbt run`で上記を実行できます。結果の確認は以下のSQLで10件のレコードが挿入されているか確認してください。

```sql
SELECT * FROM rooms;
SELECT count(*) FROM;
```

---
3. ScalikeJDBC公式ドキュメントの[transaction](http://scalikejdbc.org/documentation/transaction.html)を参考に`localTx`を使って次の動作をするメソッドをMain.scalaの中に作ってください。まだ実行はしなくていいです。

- 3.1 引数にroomIdとcustomerIdをもつ: `def persistReservation(roomId: Int, customerId: Int): Either[Exception, Int]`
  - reservationsテーブルのroom_idとcustomer_idに相当、戻り値型のEither中のExceptionはデータベース操作失敗を表し、Intはroomsテーブルの更新レコード数すなわち = 1
- 3.2 トランザクションを開始して、roomIdと一致するレコードをroomsテーブルから取得、このとき`SELECT ... FOR UPDATE`ロックを使ってください
- 3.3 前ステップ3.2でレコードが見つかれば、新しいレコードをreservationsテーブルに挿入
- 3.4 同一トランザクション内でroomsテーブルのroomIdと一致するレコードのavailableカラムをFALSEに更新
- 3.5 トランザクションを閉じてコミット

解答) 

`SELECT ... FOR UPDATE`が何であるかは、MySQLのドキュメントをじっくり読み込まないとわかりませんし、読み込んでもわからない可能性が高いです。
https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-model.html

この課題の6. で`SELECT ... FOR UPDATE`を忘れて`SELECT`にするとどうなるかを示しています。トランザクションとロックの重要性をなんとなく感じてもらえばいいと思います。
もちろん正確に理解できればよいですが、奥の深い分野なので今すぐ正確に理解できなくて良いです。

```scala
case class Room(id: Int, available: Boolean)

def persistReservation(roomId: Int, customerId: Int): Either[Exception, Int] =
  try {
    val result = DB localTx { implicit session =>
      // roomIdと一致するレコードをroomsテーブルから取得、このとき"SELECT ... FOR UPDATE"ロックを使ってください
      val roomOpt = sql"SELECT id, available FROM rooms WHERE id = ${roomId} FOR UPDATE".map { rs => Room(rs.int("id"), rs.boolean("available")) }.single().apply()
      roomOpt match {
        case None => throw new Exception(s"Room ${roomId} is not found")

        case Some(room) if (!room.available) =>
          throw new Exception(s"Customer ${customerId} could not book ${room} as it was already booked.")

        case Some(_) =>
          // 前ステップでレコードが見つかれば、新しいレコードをreservationsテーブルに挿入
          val insertCount = sql"INSERT INTO reservations (room_id, customer_id) VALUES (${roomId}, ${customerId})".update().apply()
          if (insertCount == 1) {
            // 同一トランザクション内でroomsテーブルのroomIdと一致するレコードのavailableカラムをFALSEに更新
            sql"UPDATE rooms SET available = false WHERE id = ${roomId}".update().apply()
          } else if (insertCount == 0) {
            throw new Exception(s"Failed to insert a record into `reservations` table, with roomId = ${roomId}, and customerId = ${customerId}")
          }
          else {
            throw new Exception(s"Weird Insert Count = ${insertCount}!! Failed to insert a record into `reservations` table, with roomId = ${roomId}, and customerId = ${customerId}")
          }
      }
    }
    Right(result)
  } catch {
    case e: Exception => Left(e)
  }
```

---
4. 以下の手順に従ってMain.scalaを完成させてください:

- 4.1 `def persistAll(customerId: Int)`を作って、roomId = 1から10まで`persistReservation(roomId, customerId)`を呼び出してください
- 4.2 `def persistAll(customerId: Int)`の中で、それぞれの`persistReservation`の成功/失敗の結果を`println`で表示してください
- 4.3 3つのスレッドを開始し、それぞれのスレッドに異なるcustomerIdを割り当て、`persistAll`をよびだしてください。つまり次のようなコードになります。

```scala
  def main(args: Array[String]): Unit = {
    for (customerId <- 1 to 3) {
      val thread = new Thread(() => persistAll(customerId))
      thread.start()
    }
  }
```

解答) 

```scala
package example

import scalikejdbc._
import scalikejdbc.config._

object Main {
  case class Room(id: Int, available: Boolean)

  def persistReservation(roomId: Int, customerId: Int): Either[Exception, Int] = ... //課題3. の答えなので省略

  def persistAll(customerId: Int): Unit = {
    var count = 0
    for (roomId <- 1 to 10) {
      count = roomId
      persistReservation(roomId, customerId) match {
        case Left(exception) => println(exception.getMessage)
        case Right(_) => println(s"Customer ${customerId} could successfully book the room ${roomId}")
      }
    }
  }

  def main(args: Array[String]): Unit = {
    //ログ出力が大量になるので割愛
    GlobalSettings.loggingSQLAndTime = LoggingSQLAndTimeSettings(
      enabled = false
    )

    DBs.setupAll()

    for (customerId <- 1 to 3) {
      val thread = new Thread(() => persistAll(customerId))
      thread.start()
    }
  }
}
```

実行結果はこのようになります。

```
Customer 2 could successfully book the room 1
Customer 1 could not book Room(1,false) as it was already booked.
Customer 3 could not book Room(1,false) as it was already booked.
Customer 2 could successfully book the room 2
Customer 3 could not book Room(2,false) as it was already booked.
Customer 1 could not book Room(2,false) as it was already booked.
Customer 2 could successfully book the room 3
Customer 3 could not book Room(3,false) as it was already booked.
Customer 1 could not book Room(3,false) as it was already booked.
Customer 2 could successfully book the room 4
Customer 3 could not book Room(4,false) as it was already booked.
Customer 1 could not book Room(4,false) as it was already booked.
Customer 2 could successfully book the room 5
Customer 3 could not book Room(5,false) as it was already booked.
Customer 1 could not book Room(5,false) as it was already booked.
Customer 2 could successfully book the room 6
Customer 3 could not book Room(6,false) as it was already booked.
Customer 1 could not book Room(6,false) as it was already booked.
Customer 2 could successfully book the room 7
Customer 3 could not book Room(7,false) as it was already booked.
Customer 1 could not book Room(7,false) as it was already booked.
Customer 2 could successfully book the room 8
Customer 3 could not book Room(8,false) as it was already booked.
Customer 1 could not book Room(8,false) as it was already booked.
Customer 2 could successfully book the room 9
Customer 1 could not book Room(9,false) as it was already booked.
Customer 3 could not book Room(9,false) as it was already booked.
Customer 2 could successfully book the room 10
Customer 3 could not book Room(10,false) as it was already booked.
Customer 1 could not book Room(10,false) as it was already booked.
[success] Total time: 2 s, completed Mar 30, 2020, 1:32:27 AM
```

---
5. 実験をやり直すため、以下のSQLを実行してテーブルを初期化してください。

```sql
-- FOREIGN KEYがあるのでreservationを先に消さないとエラー
DROP TABLE reservations;
DROP TABLE rooms;

CREATE TABLE `rooms` (
  `id` int NOT NULL,
  `available` BOOLEAN NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `reservations` (
  `id` int NOT NULL AUTO_INCREMENT,
  `room_id` int NOT NULL,
  `customer_id` int NOT NULL,
  PRIMARY KEY (`id`),
  FOREIGN KEY(`room_id`) REFERENCES rooms(id)
) ENGINE=INNODB DEFAULT CHARSET=utf8mb4 AUTO_INCREMENT=1;
```

解答) 実行結果は割愛します。以下のSQLで確かめてください。

```sql
SELECT * FROM rooms;
SELECT count(*) FROM rooms WHERE available = true;

SELECT * FROM reservations;
SELECT count(*) FROM reservations;
```

---
6. `def persistReservation`のなかの`SELECT ... FOR UPDATE`を、ただの`SELECT`に書き換えて何が起こるか観察します。以下の手順を実行して、その結果を貼り付けてください。

- 6.1 DataInsert.scalaを実行してroomsテーブルに10件のデータを挿入してください
- 6.2 `def persistReservation`書き換え後に再びMain.scalaを実行して、結果を貼り付けてください

解答) 6.2の`def persistReservation`の書き換え部分はこうです。

```scala
[-] val roomOpt = sql"SELECT id, available FROM rooms WHERE id = ${roomId} FOR UPDATE".map { rs => Room(rs.int("id"), rs.boolean("available")) }.single().apply()
[+] val roomOpt = sql"SELECT id, available FROM rooms WHERE id = ${roomId}".map { rs => Room(rs.int("id"), rs.boolean("available")) }.single().apply()
```

ここでMain.scalaを実行すると、今度は以下のようにエラーが発生していることがわかります。

```
Customer 3 could successfully book the room 1
Customer 3 could successfully book the room 2
Customer 3 could successfully book the room 3
01:57:20.235 [Thread-36] ERROR scalikejdbc.StatementExecutor$$anon$1 - SQL execution failed (Reason: Deadlock found when trying to get lock; try restarting transaction):

   UPDATE rooms SET available = false WHERE id = 1

01:57:20.235 [Thread-35] ERROR scalikejdbc.StatementExecutor$$anon$1 - SQL execution failed (Reason: Deadlock found when trying to get lock; try restarting transaction):

   UPDATE rooms SET available = false WHERE id = 1

Deadlock found when trying to get lock; try restarting transaction
Deadlock found when trying to get lock; try restarting transaction
...
...
...
Deadlock found when trying to get lock; try restarting transaction
Customer 1 could successfully book the room 10
[success] Total time: 2 s, completed Mar 30, 2020, 1:57:20 AM
```

そしてテーブルの中身も不正になっています。`SELECT ... FOR UPDATE`の重要性がわかると思います。

```sql
SELECT count(*) FROM rooms WHERE available = true;
-- count 0
SELECT count(*) FROM reservations;
-- count 17!!!!
```

