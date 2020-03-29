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

2. `sbt new`からScalaプロジェクトを作成して(課題[03-sql-operations.md](./03-sql-operations.md)のプロジェクトを再利用すると良いでしょう)、DataInsert.scalaファイルを作成し、
mainメソッドの中でroomsテーブルの中に10件のレコードを挿入してください

3. ScalikeJDBC公式ドキュメントの[transaction](http://scalikejdbc.org/documentation/transaction.html)を参考に`localTx`を使って次の動作をするメソッドをMain.scalaの中に作ってください。まだ実行はしなくていいです。

- 3.1 引数にroomIdとcustomerIdをもつ: `def persistReservation(roomId: Int, customerId: Int): Either[Exception, Int]`
  - reservationsテーブルのroom_idとcustomer_idに相当、戻り値型のEither中のExceptionはデータベース操作失敗を表し、Intはroomsテーブルの更新レコード数すなわち = 1
- 3.2 トランザクションを開始して、roomIdと一致するレコードをroomsテーブルから取得、このとき`SELECT ... FOR UPDATE`ロックを使ってください
- 3.3 前ステップ3.2でレコードが見つかれば、新しいレコードをreservationsテーブルに挿入
- 3.4 同一トランザクション内でroomsテーブルのroomIdと一致するレコードのavailableカラムをFALSEに更新
- 3.5 トランザクションを閉じてコミット

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

6. `def persistReservation`のなかの`SELECT ... FOR UPDATE`を、ただの`SELECT`に書き換えて何が起こるか観察します。以下の手順を実行して、その結果を貼り付けてください。

- 6.1 DataInsert.scalaを実行してroomsテーブルに10件のデータを挿入してください
- 6.2 `def persistReservation`書き換え後に再びMain.scalaを実行して、結果を貼り付けてください
