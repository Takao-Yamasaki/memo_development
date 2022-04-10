# rails db:migrateでエラーがでた
# 1.はまったこと
- NOT NULL制約を記述したマイグレーションファイルがちゃんと動作するか確認したい。
```Ruby:db/migrate/20220401031908_change_tasks_name_not_null.rb
class ChangeTasksNameNotNull < ActiveRecord::Migration[5.2]
  def change
    change_column_null :tasks, :name, false
  end
end
```
- terminalで```rails c```を起動して、Task.nameにnilを入れてみる。
```Ruby:terminal
irb(main):002:0> Task.new(name: nil).save
   (0.1ms)  begin transaction
  Task Create (1.5ms)  INSERT INTO "tasks" ("created_at", "updated_at") VALUES (?, ?)  [["created_at", "2022-04-01 03:23:07.348449"], ["updated_at", "2022-04-01 03:23:07.348449"]]
   (0.1ms)  rollback transaction
Traceback (most recent call last):
        2: from (irb):2
        1: from (irb):2:in `rescue in irb_binding'
ActiveRecord::NotNullViolation (SQLite3::ConstraintException: NOT NULL constraint failed: tasks.name: INSERT INTO "tasks" ("created_at", "updated_at") VALUES (?, ?))
```
- NOT NULL制約がない場合の挙動も確認したいと思い、マイグレーションのバージョンをひとつ戻すため、```rails db:rollback```を実行。
```Ruby:terminal
$ rails db:rollback
== 20220401031908 ChangeTasksNameNotNull: reverting ===========================
-- change_column_null(:tasks, :name, true)
   -> 0.0102s
== 20220401031908 ChangeTasksNameNotNull: reverted (0.0368s) ==================
```
- NOT NULL制約がない場合は、データベースに登録された
```Ruby:terminal
irb(main):001:0> Task.new(name: nil).save
   (0.0ms)  begin transaction
  Task Create (0.5ms)  INSERT INTO "tasks" ("created_at", "updated_at") VALUES (?, ?)  [["created_at", "2022-04-01 03:27:17.871755"], ["updated_at", "2022-04-01 03:27:17.871755"]]
   (1.1ms)  commit transaction
=> true
```
- NOT NULL制約がある場合に戻したくて、```rails db:migrate```を実行すると、```StandardError: An error has occurred, this and all later migrations canceled:```とのエラーが表示された。
```Ruby:terminal
$ rails db:migrate
== 20220401031908 ChangeTasksNameNotNull: migrating ===========================
-- change_column_null(:tasks, :name, false)
rails aborted!
StandardError: An error has occurred, this and all later migrations canceled:
```
# 2.試したこと
- まずは、マイグレーションの状態を確認。
```Ruby:terminal
$ rake db:migrate:status

database: /Users/takao.yamasaki/Documents/ruby_quick_learning/taskleaf/db/development.sqlite3

 Status   Migration ID    Migration Name
--------------------------------------------------
   up     20220331091028  Create tasks
  down    20220401031908  Change tasks name not null
```
- どうやら、データベースが初期化されていないために、起こったエラーのよう。```db:migrate:reset```を実行。
- ```rails db:rollback```でNOT NULL制約については、自動的にロールバックしたが、その後、DBに対して、直接```Task.new(name: nil).save```を実行しているため、自動的にロールバックしなかった。
```Ruby:terminal
$ rails db:migrate:reset
Dropped database 'db/development.sqlite3'
Dropped database 'db/test.sqlite3'
Created database 'db/development.sqlite3'
Created database 'db/test.sqlite3'
== 20220331091028 CreateTasks: migrating ======================================
-- create_table(:tasks)
   -> 0.0015s
== 20220331091028 CreateTasks: migrated (0.0017s) =============================

== 20220401031908 ChangeTasksNameNotNull: migrating ===========================
-- change_column_null(:tasks, :name, false)
   -> 0.0081s
== 20220401031908 ChangeTasksNameNotNull: migrated (0.0083s) ==================
```
# 3.参考
- [rails db:migrateでエラー（StandardError: An error has occurred, this and all later migrations canceled:が出た時の対処](https://qiita.com/mat827/items/b165f90a6703583b73ba)
- [【Rails】ロールバック(rollback)で何が起こっているか？schema_migrationsとは？意味と役割。UPとDOWNとは?それぞれの使い方](https://prograshi.com/framework/rails/rollback_schema-migrations/)
- [【Rails】マイグレーションファイルでchangeメソッドが使える定義一覧表](https://prograshi.com/framework/rails/change-method-in-migration/)