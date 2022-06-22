# WordPressコンテナ・MYSQLコンテナの構築

- ネットワークの作成
```
$ docker network create wordpress000net1
```

- MYSQLコンテナの作成
```
$ docker run --name mysql000ex11 -dit --net=wordpress000net1 -e MYSQL_ROOT_PASSWORD=myrootpass -e MYSQL_DATABASE=wordpress000db -e MYSQL_USER=wordpress000kun -e MYSQL_PASSWORD=wkunpass mysql --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci --default-authentication-plugin=mysql_native_password
```

- WordPressの作成
```
$ docker run --name wordpress000ex12 -dit --net=wordpress000net1 -p 8085:80 -e WORDPRESS_DB_HOST=mysql000ex11 -e WORDPRESS_DB_NAME=wordpress000db -e WORDPRESS_DB_USER=wordpress000kun -e WORDPRESS_DB_PASSWORD=wkunpass wordpress
```

# RedmineコンテナとMYSQLコンテナを作成

- ネットワークの作成
```
$ docker network create redmine000net2
```

- MYSQLコンテナの作成
```
$ docker run --name mysql000ex13 -dit --net=redmine000net2 -e MYSQL_ROOT_PASSWORD=myrootpass -e MYSQL_DATABASE=redmine000db -e MYSQL_USER=redmine000kun -e MYSQL_PASSWORD=rkunpass mysql --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci --default-authentication-plugin=mysql_native_password
```

- Redmineコンテナの作成
```
$ docker run -dit --name redmine000ex14 -dit --network redmine000net2 -p 8086:3000 -e REDMINE_DB_MYSQL=mysql000ex13 -e REDMINE_DB_DATABASE=redmine000db -e REDMINE_DB_USERNAME=redmine000kun -e REDMINE_DB_PASSWORD=rkunpass redmine
```

## RedmineコンテナとMariaDBコンテナを作成

- ネットワークを作成
```
$ docker network create redmine000net3
```

- MariaDBコンテナの作成
```
$ docker run --name mariadb000ex15 -dit --net=redmine000net3 -e MYSQL_ROOT_PASSWORD=mariarootpass -e MYSQL_DATABASE=redmine000db -e MYSQL_USER=redmine000kun -e MYSQL_PASSWORD=rkunpass mariadb --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci --default-authentication-plugin=mysql_native_password

# --character-set-server=utf8mb4 文字コード
# --collation-server=utf8mb4_unicode_ci 照合順序（データの並び替えや比較を行うための規則）
# --default-authentication-plugin=mysql_native_password 認証方式
```

- Redmineコンテナの作成と起動
```
$ docker run -dit --name redmine000ex16 --network redmine000net3 -p 8087:3000 -e REDMINE_DB_MYSQL=mariadb000ex15 -e REDMINE_DB_DATABASE=redmine000db -e REDMINE_DB_USERNAME=redmine000kun -e REDMINE_DB_PASSWORD=rkunpass redmine
```

## WordPressとMariaDBコンテナの構築
- ネットワークの作成
```
$ docker network create wordpress000net4
```

- MariaDBコンテナの作成・起動
```
$ docker run --name mariadb000ex17 -dit --net=wordpress000net4 -e MYSQL_ROOT_PASSWORD=mariarootpass -e MYSQL_DATABASE=wordpress000db -e MYSQL_USER=wordpress000kun -e MYSQL_PASSWORD=wkunpass mariadb --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci --default-authentication-plugin=mysql_native_password
```

- WordPressコンテナの作成・起動
```
$ docker run --name wordpress000ex18 -dit --net=wordpress000net4 -p 8088:80 -e WORDPRESS_DB_HOST=mariadb000ex17 -e WORDPRESS_DB_NAME=wordpress000db -e WORDPRESS_DB_USER=wordpress000kun -e WORDPRESS_DB_PASSWORD=wkunpass wordpress
```

## ホストからコンテナの中にファイルをコピーする
- Apacheコンテナを作成
```
$ docker run --name apa000ex19 -d -p 8089:80 httpd
```

- ホストからコンテナへファイルをコピー
```
$ docker cp /Users/takao.yamasaki/Documents/index.html apa000ex19:/usr/local/apache2/htdocs/
```

## コンテナからホストの中にファイルをコピーする
- コンテナは、`apa000ex19`をそのまま使用する

- ファイルを区別するために、ホストのファイルをリネームしておく
```
$ mv /Users/takao.yamasaki/Documents/index.html /Users/takao.yamasaki/Documents/index2.html
```

- コンテナからホストへファイルをコピー
```
$ docker cp apa000ex19:/usr/local/apache2/htdocs/ /Users/takao.yamasaki/Documents/index.html
```

## ボリュームマウントとバインドマウントの違い

- ボリュームマウントとは、Dockerエンジンが管理している領域内にボリュームを作成して、コンテナにマウントすること。
- バインドマウントとは、Dockerエンジンが管理していない場所に存在しているディレクトリやファイルにマウント（取り付ける）すること。

## バインドマウントしてみる
- Apacheコンテナの作成・起動
```
$ docker run --name apa000ex20 -d -p 8090:80 -v /Users/takao.yamasaki/Documents/apa_folder:/usr/local/apache2/htdocs httpd
```
- マウントしたディレクトリにindex.htmlを配置
```
$ mv /Users/takao.yamasaki/Documents/index.html /Users/takao.yamasaki/Documents/apa_folder/ 
```

## ボリュームマウントしてみる
- マウントするボリュームの作成
```
$ docker volume create apa000vol1
```
- Apacheコンテナを起動する
```
$ docker run --name apa000ex21 -d -p 8091:80 -v apa000vol1:/usr/local/apache2/htdocs httpd
```
- ボリュームの詳細情報を表示
```
$ docker volume inspect apa000vol1
[
    {
        "CreatedAt": "2022-06-13T11:45:32Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/apa000vol1/_data",
        "Name": "apa000vol1",
        "Options": null,
        "Scope": "local"
    }
]
```
- マウントされているかどうかを調べる
```
$ docker container inspect 

 "Mounts": [
            {
                "Type": "volume",
                "Name": "apa000vol1",
                "Source": "/var/lib/docker/volumes/apa000vol1/_data",
                "Destination": "/usr/local/apache2/htdocs",
                "Driver": "local",
                "Mode": "z",
                "RW": true,
                "Propagation": ""
            }
        ],
```

- 後始末を行う
	- ボリュームの状況確認
```
$ docker volume ls
DRIVER    VOLUME NAME
local     apa000vol1
local     insta_clone_ver7_bundle
local     insta_clone_ver7_mysql_data
```
	- ボリュームの削除
```
$ docker volume rm apa000vol1
```

## Docker上でのlog調査
- コンテナが何かの理由で起動しなかった場合などに、`docker logs`を使って、logを調査する
```
$ docker logs mariadb000ex15
2022-06-03 06:17:45+00:00 [Note] [Entrypoint]: Entrypoint script for MariaDB Server 1:10.8.3+maria~jammy started.
2022-06-03 06:17:45+00:00 [ERROR] [Entrypoint]: mariadbd failed while attempting to check config
	command was: mariadbd --character--set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci --default-authetication-plugin=mysql_native_password --verbose --help --log-bin-index=/tmp/tmp.JRr2Sfh1lc
	2022-06-03  6:17:45 0 [Warning] Could not open mysql.plugin table: "Table 'mysql.plugin' doesn't exist". Some options may be missing from the help text
2022-06-03  6:17:45 0 [ERROR] mariadbd: unknown variable 'character--set-server=utf8mb4'
```

## 便利なコマンド

- 現在マウントしていないボリュームを全て削除する。
```
$ docker volume prune
```

## 参考
- [【SQL Server】照合順序をざっくりわかるようになる](https://kojimanotech.com/2021/07/03/325/)
