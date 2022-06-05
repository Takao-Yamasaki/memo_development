# Docker環境の構築

## はじめてのDocker環境構築

- 環境構築には、以下のファイルが必要で、事前に突合をおこなった。
  - Dockerfile（突合済み）
  - docker-compose.yml（突合済み）
  - Gemfile
  - Gemfile.lock
  - entrypoint.sh（突合済み）

- 以下を実施して、docker-compose.ymlからイメージをビルドし、コンテナを起動した。
```
$ docker-compose up -d
```

- イメージの確認
```
$ docker image ls -a
REPOSITORY                         TAG       IMAGE ID       CREATED          SIZE
insta_clone_ver7_web               latest    70ce173642e9   48 seconds ago   1.07GB
selenium/standalone-chrome-debug   latest    aaf43463c572   8 months ago     1.11GB
mysql                              5.7.33    450379344707   13 months ago    449MB
```

- コンテナの確認
  - 全て稼働していることを確認
```
❯ docker container ls -a
CONTAINER ID   IMAGE                                     COMMAND                  CREATED              STATUS              PORTS                               NAMES
8dee776d6ed6   insta_clone_ver7_web                      "entrypoint.sh bash …"   About a minute ago   Up About a minute   0.0.0.0:3000->3000/tcp              insta_clone_ver7_web_1
1cf3a393caab   mysql:5.7.33                              "docker-entrypoint.s…"   About a minute ago   Up About a minute   0.0.0.0:3306->3306/tcp, 33060/tcp   insta_clone_ver7_db_1
e36d216879ad   selenium/standalone-chrome-debug:latest   "/opt/bin/entry_poin…"   About a minute ago   Up About a minute   0.0.0.0:4444->4444/tcp, 5900/tcp    insta_clone_ver7_chrome_1
```

- この状態でローカルホストにアクセスすると、以下のようなエラーが表示される。
  - まだ、DBを作成していないので、当たり前といえば、当たり前。
```
ActiveRecord::NoDatabaseError
We could not find your database: insta_clone_v7_development. Which can be found in the database configuration file located at config/database.yml.
```

- `database.yml`にDBの設定を記述して、以下のコマンドを実施して、DBを作成
```
$ docker-compose run web rails db:create
Creating insta_clone_ver7_web_run ... done
Created database 'insta_clone_v7_development'
Created database 'insta_clone_v7_test'
```

- その状態でローカルホストにアクセスすると、以下のようなエラーが表示される。
   - たぶん、マイグレートしなさいよというエラー。はいはい。
```
ActiveRecord::PendingMigrationError
Migrations are pending. To resolve this issue, run:
bin/rails db:migrate RAILS_ENV=development
You have 1 pending migration:
20220522113206_sorcery_core.rb
```

- コンテナ内に入る
```
$ docker-compose exec web bash
root@8dee776d6ed6:/insta_clone#
```

- 'bundle install'を実施して、Gemをインストール
```
# bundle install
```
- 'bin/rails db:migrate' を実施して、マイグレーションファイルを適用
```
# bin/rails db:migrate
== 20220522113206 SorceryCore: migrating ======================================
-- create_table(:users)
   -> 0.0284s
== 20220522113206 SorceryCore: migrated (0.0286s) =============================

Model files unchanged.
```

- 片付け
```
$ docker-compose down
```

## GemFileに変更を加えた場合

- 全て新規で作成した場合に、`docker-compose run web rails db:create`を実施すると、以下のようなエラーが表示される。
```
Could not find activestorage-validator-0.2.0 in any of the sources
Run `bundle install` to install missing gems.
ERROR:
```

### コンテナが起動してない場合
- イメージの起動とコンテナの起動をバックグラウンドで行う

```
$ docker-compose up -d
```

- `bundle install`を実施

```
$ docker-compose run web bundle install
```

- コンテナを再起動

```
$ docker-compose restart
```

### コンテナが起動している場合
- コンテナが起動している状態で、`docker-compose run web bundle install`を実行してしまうと、起動しない謎コンテナが作成されてしまう。
- `docker-compose run web bundle install`がなにやっているのか根本的によくわかっていない説。

```
$ docker container ls -a
CONTAINER ID   IMAGE                                     COMMAND                  CREATED        STATUS                    PORTS                               NAMES
db0b980a045c   insta_clone_ver7_web                      "entrypoint.sh bundl…"   16 hours ago   Exited (0) 16 hours ago                                       insta_clone_ver7_web_run_91fea5d63a17
```

- たぶんこの手順でいけるはず。
- コンテナに入る
```
$ docker-compose exec web bash
```

- 入ったら、`bundle install`を実行
  - コンテナが起動しているので、中に入れるはず。
```
# bundle install
```

- コンテナを再起動
```
$ docker-compose restart
```

## Docker上でRSpecを実行する場合

- 普通にローカル環境上で実行してしまうと、以下のようなエラーが起きる。
- そのため、一旦コンテナに入った上で、実行してあげる必要がある。

```
$ bundle exec rspec

An error occurred while loading ./spec/models/post_spec.rb.
Failure/Error: ActiveRecord::Migration.maintain_test_schema!

ActiveRecord::DatabaseConnectionError:
  There is an issue connecting with your hostname: db.

  Please check your database configuration and ensure there is a valid connection to your database.
```

- コンテナに入る
```
$ docker-compose exec web bash
```

- 無事入った後、`bundle exec rspec`を実行する
```
# bundle exec rspec
```

## Docker上でgenerateを実行する場合
- 普通にローカルで実行しても、以下のようなエラーとなる。
```
$ rails g model Comment
Could not find pagy-5.10.1 in any of the sources
Run `bundle install` to install missing gems.
```
- コンテナに入る
```
$ docker-compose exec web bash
```
- `rails g` を実行する。
```
# rails g model Comment
```


## 参考
- [docker-composeでRails 6×MySQLの開発環境を構築する方法](https://tmasuyama1114.com/docker-compose-rails6-mysql-development/)
- [dockerでGemfile(gem追加)更新したら後、データベースが消えた、gemが反映しない](https://qiita.com/gyu_outputs/items/1cc1903db802daf0fdbc)
- [Docker上でRspecを実行するとMySQLに接続できない](https://teratail.com/questions/286284)
