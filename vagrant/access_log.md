# アクセスログの分析 

# 1.やりたいこと
- 実務では、アクセスログによる調査が実施される。
- Nginxのアクセスログからステータスコード200のみを絞り、hostでソート順に並べたい。
- Nginxのインストールは、[【初心者用】Vagrantの環境構築(Mac)](https://qiita.com/takao_yamasaki/items/ab72af703b09db1cbe3e)を参考に。

# 2.やったこと
## 2.1 下準備
- 前回作成した自作コマンドで、Vagrantを起動。
    - 自作コマンドについては、[【初心者用】Vagrantの自作コマンドを作成](https://qiita.com/takao_yamasaki/items/37c6b314a95b7ae9a458)を参考に。
```
$ deploy-vagrant.sh up
```
- 前回作成した自作コマンドで、SSH接続
```
$ deploy-vagrant.sh ssh
```
## 2.2 Nginxのアクセスログの確認
- Nginxのログは /var/log/nginx 配下にある。lessコマンドで確認してみる。
```
$ less /var/log/nginx

合計 16
drwxr-xr-x.  2 root  root  123  3月  3 04:21 ./
drwxr-xr-x. 10 root  root 4096  3月  3 01:08 ../
-rw-r-----   1 nginx adm     0  2月 27 03:15 access.log
-rw-r-----.  1 nginx adm   435  2月 26 12:00 access.log-20220227
-rw-r-----   1 nginx adm     0  3月  3 04:21 error.log
-rw-r-----   1 nginx adm   940  2月 27 03:15 error.log-20220227.gz
-rw-r-----   1 nginx adm   685  3月  3 04:21 error.log-20220303
```
## 2.3 サンプルのaccess.logファイルを作成
- 今回は、サンプルファイル'access.log'を作成して、アクセスログの加工を行なっていく。
```
$ touch access.log
```
- サンプルのログを貼っていく
```
$ vi access.log
```
- こんな感じで`access.log`を作成した
```
$ cat access.log
[nginx] time:2022-01-03T17:58:33+09:00  server_addr:10.15.0.5   host:10.15.0.2  method:POST     reqsize:385     uri:/wp-cron.php?doing_wp_cron=1641200313.7922360897064208984375      query:doing_wp_cron=1641200313.7922360897064208984375   status:200      size:31 referer:https://blog.adachin.me/wp-cron.php?doing_wp_cron=1641200313.7922360897064208984375   ua:WordPress/5.8.2; https://blog.adachin.me  forwardedfor:-   reqtime:0.001   apptime:0.004
[nginx] time:2022-01-03T17:58:34+09:00  server_addr:10.15.0.5   host:185.191.171.16     method:GET      reqsize:303     uri:/archives/10835   query:q=/archives/10835&        status:200      size:26491      referer:-       ua:Mozilla/5.0 (compatible; SemrushBot/7~bl; +http://www.semrush.com/bot.html)        forwardedfor:-  reqtime:2.278   apptime:2.280
[nginx] time:2022-01-03T17:59:06+09:00  server_addr:10.15.0.5   host:10.15.0.2  method:POST     reqsize:385     uri:/wp-cron.php?doing_wp_cron=1641200346.7894539833068847656250      query:doing_wp_cron=1641200346.7894539833068847656250   status:200      size:31 referer:https://blog.adachin.me/wp-cron.php?doing_wp_cron=1641200346.7894539833068847656250   ua:WordPress/5.8.2; https://blog.adachin.me  forwardedfor:-   reqtime:0.001   apptime:0.004
[nginx] time:2022-01-03T17:59:08+09:00  server_addr:10.15.0.5   host:147.92.153.18      method:GET      reqsize:356     uri:/archives/tag/h2o/page/2  query:q=/archives/tag/h2o/page/2&       status:200      size:20008      referer:-       ua:Mozilla/5.0 (compatible; Linespider/1.1; +https://lin.ee/4dwXkTH)  forwardedfor:-  reqtime:2.834   apptime:2.832
[nginx] time:2022-01-03T17:59:06+09:00  server_addr:10.15.0.5   host:10.15.0.10  method:POST     reqsize:385     uri:/wp-cron.php?doing_wp_cron=1641200346.7894539833068847656250      query:doing_wp_cron=1641200346.7894539833068847656250   status:200      size:31 referer:https://blog.adachin.me/wp-cron.php?doing_wp_cron=1641200346.7894539833068847656250   ua:WordPress/5.8.2; https://blog.adachin.me  forwardedfor:-   reqtime:0.001   apptime:0.004
[nginx] time:2022-01-03T17:59:08+09:00  server_addr:10.15.0.10   host:147.92.153.18      method:GET      reqsize:356     uri:/archives/tag/h2o/page/2  query:q=/archives/tag/h2o/page/2&       status:200      size:20008      referer:-       ua:Mozilla/5.0 (compatible; Linespider/1.1; +https://lin.ee/4dwXkTH)  forwardedfor:-  reqtime:2.834   apptime:2.832
[nginx] time:2022-01-03T17:59:08+09:00  server_addr:10.15.0.11   host:147.92.153.18      method:GET      reqsize:356     uri:/archives/tag/h2o/page/2  query:q=/archives/tag/h2o/page/2&       status:200      size:20008      referer:-       ua:Mozilla/5.0 (compatible; Linespider/1.1; +https://lin.ee/4dwXkTH)  forwardedfor:-  reqtime:2.834   apptime:2.832
[nginx] time:2022-01-03T17:59:08+09:00  server_addr:10.15.0.11   host:147.92.153.18      method:GET      reqsize:356     uri:/archives/tag/h2o/page/2  query:q=/archives/tag/h2o/page/2&       status:200      size:20008      referer:-       ua:Mozilla/5.0 (compatible; Linespider/1.1; +https://lin.ee/4dwXkTH)  forwardedfor:-  reqtime:2.834   apptime:2.832
[nginx] time:2022-01-03T17:59:08+09:00  server_addr:10.15.0.22   host:147.92.153.18      method:GET      reqsize:356     uri:/archives/tag/h2o/page/2  query:q=/archives/tag/h2o/page/2&       status:200      size:20008      referer:-       ua:Mozilla/5.0 (compatible; Linespider/1.1; +https://lin.ee/4dwXkTH)  forwardedfor:-  reqtime:2.834   apptime:2.832
```

## 2.4 アクセスログからステータスコード200のみを絞る
- とりあえず、awkコマンドを使ってみる。
```
$ cat access.log | awk '{print $9}'
status:200
status:200
status:200
status:200
status:200
status:200
status:200
status:200
status:200
```
- `access.log`中、今回抽出したい対象の`status:200`は、`$9`（9フィールド目）にあることがわかった。
- そのため、awkの抽出条件としては、`'$9=="status:200"`となる。
- 一旦、`{print $0}`でレコード全体を抽出する。
```
$ cat access.log | awk '$9=="status:200" {print $0}'
[nginx] time:2022-01-03T17:58:33+09:00  server_addr:10.15.0.5   host:10.15.0.2  method:POST     reqsize:385     uri:/wp-cron.php?doing_wp_cron=1641200313.7922360897064208984375      query:doing_wp_cron=1641200313.7922360897064208984375   status:200      size:31 referer:https://blog.adachin.me/wp-cron.php?doing_wp_cron=1641200313.7922360897064208984375   ua:WordPress/5.8.2; https://blog.adachin.me  forwardedfor:-   reqtime:0.001   apptime:0.004
[nginx] time:2022-01-03T17:58:34+09:00  server_addr:10.15.0.5   host:185.191.171.16     method:GET      reqsize:303     uri:/archives/10835   query:q=/archives/10835&        status:200      size:26491      referer:-       ua:Mozilla/5.0 (compatible; SemrushBot/7~bl; +http://www.semrush.com/bot.html)        forwardedfor:-  reqtime:2.278   apptime:2.280
[nginx] time:2022-01-03T17:59:06+09:00  server_addr:10.15.0.5   host:10.15.0.2  method:POST     reqsize:385     uri:/wp-cron.php?doing_wp_cron=1641200346.7894539833068847656250      query:doing_wp_cron=1641200346.7894539833068847656250   status:200      size:31 referer:https://blog.adachin.me/wp-cron.php?doing_wp_cron=1641200346.7894539833068847656250   ua:WordPress/5.8.2; https://blog.adachin.me  forwardedfor:-   reqtime:0.001   apptime:0.004
[nginx] time:2022-01-03T17:59:08+09:00  server_addr:10.15.0.5   host:147.92.153.18      method:GET      reqsize:356     uri:/archives/tag/h2o/page/2  query:q=/archives/tag/h2o/page/2&       status:200      size:20008      referer:-       ua:Mozilla/5.0 (compatible; Linespider/1.1; +https://lin.ee/4dwXkTH)  forwardedfor:-  reqtime:2.834   apptime:2.832
[nginx] time:2022-01-03T17:59:06+09:00  server_addr:10.15.0.5   host:10.15.0.10  method:POST     reqsize:385     uri:/wp-cron.php?doing_wp_cron=1641200346.7894539833068847656250      query:doing_wp_cron=1641200346.7894539833068847656250   status:200      size:31 referer:https://blog.adachin.me/wp-cron.php?doing_wp_cron=1641200346.7894539833068847656250   ua:WordPress/5.8.2; https://blog.adachin.me  forwardedfor:-   reqtime:0.001   apptime:0.004
[nginx] time:2022-01-03T17:59:08+09:00  server_addr:10.15.0.10   host:147.92.153.18      method:GET      reqsize:356     uri:/archives/tag/h2o/page/2  query:q=/archives/tag/h2o/page/2&       status:200      size:20008      referer:-       ua:Mozilla/5.0 (compatible; Linespider/1.1; +https://lin.ee/4dwXkTH)  forwardedfor:-  reqtime:2.834   apptime:2.832
[nginx] time:2022-01-03T17:59:08+09:00  server_addr:10.15.0.11   host:147.92.153.18      method:GET      reqsize:356     uri:/archives/tag/h2o/page/2  query:q=/archives/tag/h2o/page/2&       status:200      size:20008      referer:-       ua:Mozilla/5.0 (compatible; Linespider/1.1; +https://lin.ee/4dwXkTH)  forwardedfor:-  reqtime:2.834   apptime:2.832
[nginx] time:2022-01-03T17:59:08+09:00  server_addr:10.15.0.11   host:147.92.153.18      method:GET      reqsize:356     uri:/archives/tag/h2o/page/2  query:q=/archives/tag/h2o/page/2&       status:200      size:20008      referer:-       ua:Mozilla/5.0 (compatible; Linespider/1.1; +https://lin.ee/4dwXkTH)  forwardedfor:-  reqtime:2.834   apptime:2.832
[nginx] time:2022-01-03T17:59:08+09:00  server_addr:10.15.0.22   host:147.92.153.18      method:GET      reqsize:356     uri:/archives/tag/h2o/page/2  query:q=/archives/tag/h2o/page/2&       status:200      size:20008      referer:-       ua:Mozilla/5.0 (compatible; Linespider/1.1; +https://lin.ee/4dwXkTH)  forwardedfor:-  reqtime:2.834   apptime:2.832
```
- 抽出対象に`$3`と`$9`を指定し、`status200_log.txt`として出力
```
$ cat access.log | awk '$9=="status:200" {print $3, $9 > "status200_log.txt"}'
```
- 出力した`status200_log.txt`の中身を確認する
```
$ cat cat status200_log.txt
server_addr:10.15.0.5 status:200
server_addr:10.15.0.5 status:200
server_addr:10.15.0.5 status:200
server_addr:10.15.0.5 status:200
server_addr:10.15.0.5 status:200
server_addr:10.15.0.10 status:200
server_addr:10.15.0.11 status:200
server_addr:10.15.0.11 status:200
server_addr:10.15.0.22 status:200
```

## 2.5 hostでソート順に並べる
- 先頭列に重複カウントを出力したいので、`uniq`コマンドを使って重複カウントを行い、`fixlog.txt`に吐き出す
```
$ uniq -c status200_log.txt > fixlog.txt
```
- `fixlog.txt`の中身を確認すると、無事重複カウントができている。
```
$ cat fixlog.txt
      5 server_addr:10.15.0.5 status:200
      1 server_addr:10.15.0.10 status:200
      2 server_addr:10.15.0.11 status:200
      1 server_addr:10.15.0.22 status:200
```
- あとは、重複回数によって、降順にソートしたい。
```
$ sort -r fixlog.txt
      5 server_addr:10.15.0.5 status:200
      2 server_addr:10.15.0.11 status:200
      1 server_addr:10.15.0.22 status:200
      1 server_addr:10.15.0.10 status:200
```
- 無事ソートすることができた。
```
$ cat fixlog.txt
      5 server_addr:10.15.0.5 status:200
      1 server_addr:10.15.0.10 status:200
      2 server_addr:10.15.0.11 status:200
      1 server_addr:10.15.0.22 status:200
```

# 3.参考
- [awkコマンドの基本](https://qiita.com/yamazon/items/563af1b485ff413d381f)
- [とほほのAWK入門](https://www.tohoho-web.com/ex/awk.html)
- [sortコマンドについて詳しくまとめました 【Linuxコマンド集】](https://eng-entrance.com/linux-command-sort)
- [Linux：uniqコマンドで重複する行をカウントする方法](https://motamemo.com/linux/linux-tips/uniq-count-lines/)
- [IPアドレスをsortコマンドで並び替える](https://ex1.m-yabe.com/archives/4506)