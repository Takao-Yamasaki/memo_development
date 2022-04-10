# シェルスクリプトの作成

## 1.はじめに
webサーバー(nginx)は常に動作しないと、サイトにアクセスできない。
これを防ぐため、シェルスクリプトで、常に検知するようにしたい。

## ２.実行手順
まず、プロセス監視用のシェルスクリプトを作成する。
```shell:
#!/bin/sh

#監視するプロセス名を定義する
PROCESS_NAME=nginx

#監視するプロセスが何個起動しているかカウントする
count=`pgrep $PROCESS_NAME | wc -l`

#監視するプロセス数を出力する
echo "nginx process $count"

#プロセス数が0であれば、nginxを起動する
if [ "$count" = 0 ]
then
systemctl start nginx
echo "nginx is not running"
echo "nginx is running"
else
echo "nginx is running"
fi
```
最終的にcronで5分に一回検知するよう設定する

## 3.参考記事
### 【シェルスクリプト】
- [初心者向けシェルスクリプトの基本コマンドの紹介](https://qiita.com/zayarwinttun/items/0dae4cb66d8f4bd2a337)
- [nginxを監視するシェルスクリプト](http://fanblogs.jp/ekusuy69/archive/4/0)
- [Nginxでstatusの確認など 基本コマンド15選](https://freelance.techcareer.jp/articles/11270/)
- [Vagrantでnginx](https://ideal-reality.com/computer/server/vagrant-nginx/)
- [sudo、yumって何？Linux基本コマンドがわからないとAWSは理解出来ない！](https://denblog.info/archives/1732)
- [vagrantの初期パスワード](https://www.suzu6.net/posts/191-vagrant-init-passwd/)
- [vagrantのSSH接続でrootに切り替える](https://kamonohashi.club/2019/06/26/25/開発環境/)
### 【cron】
- [【初心者向け】cronを利用したプロセス監視](https://qiita.com/shark_No95/items/e3af590e87879acc273e)
- [実践で使えるcrontabの書き方・設定方法をわかりやすく説明する【Ubuntu】](https://self-development.info/実践で使えるcrontabの書き方・設定方法をわかりやす/)
- [【入門】cron（クロン）設定・書き方の基本](https://www.kagoya.jp/howto/it-glossary/server/cron/)