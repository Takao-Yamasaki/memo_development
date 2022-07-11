# Docker composeを使って、Webアプリの開発環境を構築する
## Dockerfileの作成
- 複数コンテナを操作するには、docker composeが便利
- 通常デスクトップ版をダウンロードした時点で、docker-composeも使用できるようになっている

- バージョン確認をして、使えるか確認
```
$ docker-compose --version
```

- ビルドコンテキストとなる`product-register`を作成、直下に移動する
```
$ mkdir product-register && cd product-register
```

- Dockerfileを作成して、編集
```
# ベースとなるイメージを選択
FROM ruby:2.5

# 必要なパッケージをインストール
RUN apt-get update && apt-get install -y \
    build-essential \
    libpq-dev \
    nodejs \
    postgresql-client \
    yarn

# コンテナ側にフォルダを自動で作成して、コマンドを実行
WORKDIR /product-register

# コンテナ側にGemfileとGemfile.lockをコピー
COPY Gemfile Gemfile.lock /product-register

# bundle installを実行する
RUN bundle install
```

- Gemfileにrailsのバージョンを記載する
```Gemfile
source 'https://rubygems.org'
gem 'rails', '~>5.2'
```

- イメージを作成する
```
docker build .
```

## docker compose
- イメージを作成して、コンテナを起動する
- 以下の場合だと、`docker compose`を使用するのが便利
  - runコマンドが長くなってしまう
  - 複数コンテナを操作する必要がある
- docker composeを使用するには、`docker-compose.yml`を記述する必要がある

- イメージがない場合はイメージを作成して、既存のイメージがある場合には既存のイメージを利用して、コンテナを起動する
```
$ docker compose up -d
```

- dockerコンテナの中に入る
```
$ docker compose exec web bash
$ docker exec -it product-register-web-1 bash
```

## Railsのセットアップ
- 今、Railsのパッケージをインストールしているだけなので、Railsのプロジェクトを新規作成する
- コンテナ内で以下を実行
  - `rails new`すると、`bundle install`されてしまい、上書きされるので、スキップする
  -   Dockerfileに記述しているので、実行不要 
  - --forceで`Gemfile.lock`を上書きする
```
$ rails new . --force --force --database=postgresql --skip-bundle
```

- コンテナ内でサーバーを起動してみる
  - Gemfileを更新した場合は、bundle installする必要がある。
  - bundle installは、Dockerfileに記述しており、コンテナとイメージを新しく作成する必要がある。
```
$ rails s -b 0.0.0.0
Could not find gem 'pg (>= 0.18, < 2.0)' in any of the gem sources listed in your Gemfile.
Run `bundle install` to install missing gems.
```
- コンテナを削除（ただし、イメージは残る）
```
$ docker compose down
```

- イメージをビルドし直す必要があるため、'--build'を指定する
```
$ docker compose up --build -d
```

- コンテナ内でサーバーを起動する
  - `0.0.0.0`については、ローカルという意
```
$ rails s -b 0.0.0.0
```

## docker-compose.ymlにDB部分を追記
- DBのコンテナをを作成していないので、当然エラーとなる
```
$ docker compose exec web bash
root@62bdbf466ba2:/product-register# rails db:create
could not connect to server: No such file or directory
        Is the server running locally and accepting
        connections on Unix domain socket "/var/run/postgresql/.s.PGSQL.5432"?
Couldn't create 'product-register_development' database. Please check your configuration.
rails aborted!
```

- `database.yml`と`docker-compose.yml`を以下のように編集

```config/database.yml
default: &default
  adapter: postgresql
  encoding: unicode
  host: db
  user: postgres
  port: 5432
  password:<%= ENV.fetch("DATABASE_PASSWORD") %>
```
```docker-compose.yml
# dockeer composeのバージョンで、ほとんど3でOK
version: '3'

volumes:
  db-data:

services:
  web:
    # Dockerfileを元にイメージを作成するので、カレントディレクトリを指定
    build: .
    # <host_port>:<container_port>
    ports:
      - '3000:3000'
    # ホストとコンテナをマウントする
    # ホスト側は相対パスを指定
    volumes:
      - '.:/product-register'
    environment:
      - 'DATABASE_PASSWORD=postgres'
    # コマンドだと、'-it'で指定する箇所
    # 出力結果を綺麗にする
    tty: true
    # 入力チャネルを開く
    stdin_open: true
    depends_on:
      - db
    links:
      - db

  db:
  # イメージを取得する
    image: postgres
    volumes:
      - 'db-data:/var/lib/postgresql/data'    
```

## Railsアプリの作成
- １つのアプリを１つのコンテナに入れるのが鉄則
- 基本的に、アプリとDBのコンテナそれぞれ立てる
- ホスト上に`db-data`というdockerボリュームを作成する
  - あくまで仮想マシン上で作られたボリュームであるため、`/var/lib/docker/volumes/product-register_db-data`には移動できない

```docker-compose.yml
# dockeer composeのバージョンで、ほとんど3でOK
version: '3'

volumes:
  db-data:

services:
  web:
    # Dockerfileを元にイメージを作成するので、カレントディレクトリを指定
    build: .
    # <host_port>:<container_port>
    ports:
      - '3000:3000'
    # ホストとコンテナをマウントする
    # ホスト側は相対パスを指定
    volumes:
      - '.:/product-register'
    environment:
      - 'DATABASE_PASSWORD=postgres'
    # コマンドだと、'-it'で指定する箇所
    # 出力結果を綺麗にする
    tty: true
    # 入力チャネルを開く
    stdin_open: true
    depends_on:
      - db
    links:
      - db

  db:
  # イメージを取得する
    image: postgres
    volumes:
      - 'db-data:/var/lib/postgresql/data'
  # 環境変数を追加
    environment:
      - 'POSTGRES_USER=postgres'
      - 'POSTGRES_PASSWORD=postgres'    
```

- コンテナを起動して、コンテナ内に入る
```
$ docker compose up -d
$ docker compose exec web bash 
```
- DBの作成
```
$ rails db:create
```
- スカフォルドの作成
```
$ rails g scaffold product name:string price:integer vender:string
```
- マイグレートする
```
$ rails db:migrate
```
- サーバーをローカルで起動
```
$ rails s -b 0.0.0.0
```

# CI/CDパイプラインを構築する
- テスト環境上にDockerで環境構築して、テストが通るか検証できる仕組みがCI
  - 具体的には、'travis-ci.com'のサーバー上にDocker環境を構築して、テストが通るか検証している
  - GitHubへのpushで以下のymlがキックされる。

```travis.yml
# 権限の設定
sudo: required

# Dockerを使う
services: docker

# コンテナを起動
before_install:
  - docker-compose up --build -d

# DBの準備
# テストの実行
script:
  - docker-compose exec --env 'RAILS_ENV=test' web rails db:create
  - docker-compose exec --env 'RAILS_ENV=test' web rails db:migrate
  - docker-compose exec --env 'RAILS_ENV=test' web rails test

```

## herokuの設定
- 本番環境では、コンテナのDBを使用せず、ホストのDBを使用することが多い。
- まず、heroku上でアプリを作成する
  - herokuのアプリ名はそのままurlになるので、一意のものになるようにする
- 今回はherokuにホストするので、`heroku postgres`を追加する
- Settingsの`config vars`を確認する（ここに本番環境のDB情報がある）
- その下に`SECRET_KEY_BASE`を追加する（rails独自のもの）
  - valueはmaster.keyの値をコピーして貼り付ける
  - command + pで探す。めちゃ便利。
- デフォルトの`production`をコメントアウトし、コメントアウトになっている以下のコードを解除する
```database.yml
production:
    url: <%= ENV['DATABASE_URL'] %>
```

- herokuのアプリとgithubを連携する
  - HerokuのDeployを選択し、以下のように設定する
    - `Deployment method`に`GutHub`を選択
    - `App connected to GitHub`でGithubのレジストリを選択
    - `Automatic deploys`で`master`を選択したうえで` Wait for CI to pass before deploy`をチェック

## `.travis.yml`にHerokuへのデプロイを記述する
### 1.`.travis.yml`にデプロイ用の処理を記述する
  - 以下の手順を`.travis.yml`に記述していく
  - 1.HerokuのDockerレジストリにログイン
    - アプリ作成時にDockerレジストリが作成される
  - 2.本番環境用のDockerfileをビルド（image）を作成
  - 3.HerokuのDockerレジストリに本番環境用のimageをpush
  - 4.アプリ起動
  
```.travis.yml
  # 権限の設定
sudo: required

# Dockerを使う
services: docker

# コンテナを起動
before_install:
  - docker-compose up --build -d
  # 1.HerokuのDockerレジストリにログイン
  - docker login -u "$HEROKU_USERNAME" -p "$HEROKU_API_KEY" registry.heroku.com

# DBの準備
# テストの実行
script:
  - docker-compose exec --env 'RAILS_ENV=test' web rails db:create
  - docker-compose exec --env 'RAILS_ENV=test' web rails db:migrate
  - docker-compose exec --env 'RAILS_ENV=test' web rails test

deploy:
  provider: script
  script:
    # 2.本番環境用のDockerfileをビルド（imageを作成）
    docker build -t registry.heroku.com/$HEROKU_APP_NAME/web -f Dockerfile.prod .;
    
    # 3.HerokuのDockerレジストリに本番環境用のimageをpush
    docker push registry.heroku.com/$HEROKU_APP_NAME/web;
    
    # 4.アプリ起動(コンテナを立てる、dm:migrateする)
    heroku run --app $HEROKU_APP_NAME rails db:migrate;
  # デプロイはmasterにmergeされたときだけ
  on:
    branch: master

```

### 2.Macにherokuをインストールしてtokenを取得
- Heroku CLIをコマンドでダウンロード
  - https://devcenter.heroku.com/ja/articles/heroku-cli
```
$ ​brew tap heroku/brew && brew install heroku
```
- 今回は、herokuのtokenを使ってログインをするので、herokuのtokenを次のコマンドを使って取得する
```
$ heroku authorizations:create
```

### 3.TravisCIとHerokuに環境変数を設定
#### TravisCIに環境変数を設定する
- `More Options`の`Settings`から`Environment Variables`を設定していく
  - `HEROKU_USERNAME`に`_`
  - `HEROKU_API_KEY`
  - `HEROKU_APP_NAME`
#### Herokuに環境変数を設定する
- 今回は、docker-composeは使わないので、herokuにDBの環境変数を設定する必要がある
- `Settings`の`Config Vars`に追記する
  - `DATABASE_PASSWORD`に`postgres`  
### 4.本番環境用`Dockerfile.prod`を作成
- 本番環境用では、不要なパッケージとかもあるので、別途本番環境用のDockerfileを用意する
- 本番環境の場合は、本番環境にコードをコピーしてきて、本番環境内で全て完結させるのが一般的

```Dockerfile.prod
# ベースとなるイメージを選択
FROM ruby:2.5

# 必要なパッケージをインストール
RUN apt-get update && apt-get install -y \
    build-essential \
    libpq-dev \
    nodejs \
    postgresql-client \
    yarn

# コンテナ側にフォルダを自動で作成して、コマンドを実行
WORKDIR /product-register

# コンテナ側にGemfileとGemfile.lockをコピー
COPY Gemfile Gemfile.lock /product-register/

# bundle installを実行する
RUN bundle install

----------以下を追記---------------------------------
# カレントディレクトリにあるコードファイルを全てコンテナにコピー
COPY ..

# サーバーを起動
CMD ["rails", "s"]
```

- あとはGitにPushして、テスト環境上（travis.ci上）で無事ビルドされるか確認する
