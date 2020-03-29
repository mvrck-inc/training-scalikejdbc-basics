## この課題で身につく能力

QueryDSLとsql”"の違いを把握している、Query DSLは使いこなせなくてよい


## 課題

以下の課題は口頭試問です。口頭試問が終わった後のメモをこのファイルに貼り付けて、Pull Requestを送る形で提出してください。

1. 口頭試問) ScalikeJDBC公式ドキュメントの[Query DSL](http://scalikejdbc.org/documentation/query-dsl.html)のページをみて`sql"..."`とQuery DSLの違いについて答えてください。

解答) 一言でいうと、`sql"..."`の方はSQL分を全て文字列として表現して、Query DSLは`select.from(...).where(...)`のようになっていてSQL文を文字列ではなくメソッド呼び出しのチェーンで表現します。

そしてQuery DSLは型安全です。発行されるSQL文に文法的間違いが起きる可能性がより少なくなります(可能性ゼロかどうかは調べていませんごめんなさい)。どういうことかもう少し說明しましょう。
`sql"..."`の場合は`sql"selelect ffrom ..."`などとタイポをやらかしてしまう可能性があります。この場合、タイポに気づくのは実際にSQL文を実行しようとしたときに実行時エラーとして検出されます。
一方Query DSLは`selelect.ffrom`などとすれば、コンパイルエラーになって実行を待たずにエラーを検出できます。これが型安全なQuery DSLのメリットです。

では安全だから必ずQuery DSLを使うべき！かというと、そうでもありません。Query DSLの書き心地は微妙に素のSQLと違うので、いろいろな要因を考慮して決めましょう。唯一絶対の正解はありません。

ちなみに、`sql"..."`の方は型安全ではなくてもSQL Injectionに対しては安全です。こちらでかいせつされています。

http://scalikejdbc.org/documentation/sql-interpolation.html

> Don’t worry, this code is safely protected from SQL injection attacks. ${id} will be a place holder.