# XserverへのWordPressの導入方法について

## SSH設定
- 以下のサイトを参考に、公開鍵認証用のキーペアを作成する。
- [SSH設定]https://www.xserver.ne.jp/manual/man_server_ssh.php

## SSHでサーバーにログイン
- ダウンロードしたキーペアを移動
```
$ mv ~/Desktop/xs258975.key ~/.ssh/
```

- .sshに移動
```
$ cd ~/.ssh/
```

- キーペアの名前の変更
```
$ mv xs258975.key id_xserver_rsa
```

- パーミッションの変更
```
$ chmod 700 ~/.ssh
```
```
$ chmod 600 ~/.ssh/id_xserver_rsa
```

## ログイン
- ssh接続すると、以下のようにパスワードを求められるので、xserverで設定したパスコードを入力
```
$ ssh -l xs258975 -i id_xserver_rsa xs258975.xsrv.jp -p 10022
Enter passphrase for key 'id_xserver_rsa':  
```
- 無事ログインすることができた。

## ログインの簡略化
```
$ vim ~/.ssh/config
```

```
Host star
  HostName xs258975.xsrv.jp
  Port 10022
  User xs258975
  IdentityFile ~/.ssh/id_xserver_rsa
  ServerAliveInterval 60
```

-  以下のコマンドでssh接続が可能
```
$ ssh star
```

- [エックスサーバーにssh接続する方法（ターミナル利用）](https://qiita.com/ryo2132/items/38b5a93b3df476dd2b44)
- [Linux(CentOS)のsudoコマンドでsudoersファイル内にはありません / xxx is not in the sudoers file. が出てくる](https://windows-podcast.com/sundayprogrammer/archives/2609)

## Xserverでのドメインの設定方法
以下のサイトを参考に作業を進める。
- [お名前ドットコムで取得したドメインをエックスサーバーで使うための設定方法](https://wp-exp.com/blog/domein-xserver/)



