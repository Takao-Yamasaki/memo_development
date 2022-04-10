# コマンドの作成

# 1. やりたいこと
- Vagrantは、Vagrantfileがあるディレクトリに移動して、`vagrant suspend`(停止)、`vagrant status`(確認) 、`vagrant up`(起動)などを実行するのが一般的。
- Macのホームディレクトリからシェルスクリプトで引数を指定して、どのディレクトリでも動作できるようにしたい。

# 2. 内容
## 2.1 要件
- Macからシェルスクリプトを叩くとヘルプを出すようにする
- 引数でvagrantのオプションを実行できるようにする

## 2.2 実装

- ルート直下にcommandディレクトリを作成
```
$ mkdir ~/command 
```
- `deploy-vagrant.ssh`を作成
```
$ touch deploy-vagrant.ssh
```
- エディタで`deploy-vagrant.ssh`を編集する
```shell:deploy_vagrant.ssh
#!/bin/sh

#help関数の作成
help() {
  echo "下記のようにstatusを指定して実行してください。(例) ./deploy-vagrant.sh  status,up,suspend"
}

#vagrantのディレクトリに移動
cd ~/vagrant/dev

#第一引数$1で場合分け
if [ "$1" = "status" ]; then  
  vagrant status
elif [ "$1" = "up" ]; then
  #vagrantで起動
  vagrant up
  #sshでログイン
  vagrant ssh
elif [ "$1" = "suspend" ]; then
  vagrant suspend
else
  #help関数の呼び出し
  help
fi
```
- こちらの書き方のほうがスマート。柔軟性がある。
```shell:deploy_vagrant.ssh
#!/bin/sh

#help関数の作成
help() {
  echo "下記のようにstatusを指定して実行してください。(例) ./deploy-vagrant.sh  status,up,suspend"
}

#vagrantのディレクトリに移動
cd ~/vagrant/dev

# 引数の数が1個でない場合、help関数を表示し、終了する
# $# 引数の総数
# -ne 異なる
if [ $# -ne 1 ]; then
  help
  exit
# それ以外であれば、第一引数を取得し、vagrantコマンドとして実行
else
  vagrant $1
fi
```
- 権限を変更
    - 実行権限を付与する
```
$ chmod 777 ~/command/deploy-vagrant.ssh
```
- このままではコマンドとして使えないので、pathを通す必要がある。
    - `~/.bash_profile`にpathを追記する。
```
$ vi ~/.bash_profile
```
```shell:.bash_profile
$ export PATH=$PATH:~/command/
```
- `.bash_profile`を更新する
```
$ source ~/.bash_profile
```
- pathが通っているか確認する。
    - 登録した自分のパスが表示されたら、OK
```
$ echo $PATH
```
- 動作確認
```shell:$ deploy-vagrant.sh suspend
$ deploy-vagrant.sh suspend
==> default: Saving VM state and suspending execution...
```
```shell:$ deploy-vagrant.sh up
$ deploy-vagrant.sh up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Checking if box 'centos/7' version '2004.01' is up to date...
==> default: Resuming suspended VM...
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
==> default: Machine booted and ready!
==> default: Machine already provisioned. Run `vagrant provision` or use the `--provision`
==> default: flag to force provisioning. Provisioners marked to run always will still run.
Last login: Thu Mar  3 05:41:51 2022 from 10.0.2.2
```
```shell:$ deploy-vagrant.sh status
$ deploy-vagrant.sh status
Current machine states:

default                   saved (virtualbox)

To resume this VM, simply run `vagrant up`.
```
```shell:$ deploy-vagrant.sh
$ deploy-vagrant.sh
下記のようにstatusを指定して実行してください。(例) ./deploy-vagrant.sh  status,up,suspend
```
# 3. 参考
- [ターミナルで使える自作コマンドを作る](https://qiita.com/saki-engineering/items/57956af61c191b0e3282)
- [初心者向けシェルスクリプトの基本コマンドの紹介](https://qiita.com/zayarwinttun/items/0dae4cb66d8f4bd2a337)
- [MacでPATHを通す](https://qiita.com/nbkn/items/01a11392921119fa0153)
- [引数を処理する](https://shellscript.sunone.me/parameter.html)