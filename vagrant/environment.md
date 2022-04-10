# Vagrantの環境構築

## 1.はじめに
Vagrant（ベイグラント）とは、仮想環境を構築するためのツール。
※ VirtualBoxが既にインストールされていることが前提。

## 2.実施手順
### 2.1 Vagrantをインストール
- まずは、インストール用ディレクトリを作成する
```shell:ディレクトリの作成
$ mkdir -p ~/vagrant/dev
```

- [Vagrantの公式サイト](https://www.vagrantup.com/downloads.html)に、Vagrantをダウンロードするコマンドが記載されているので、参考にする。
![vagrant.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1159639/fb0cb79c-aaa1-6177-ece1-a2d3d7debc2c.png)

- `~/vargrant/dev`配下に移動して、以下のとおり、Vagrantをインストールする。
    - しばらく時間が掛かるので、あわてず待つ。
```
$ brew install vagrant
```
- インストールが終わったら、Vagrantがインストールされたか確認する。
    - バージョン確認のため、以下のコマンドで確かめる。
```
$ vagrant --version
```
### 2.2 centos7をインストール
- 今回は`centos/7`のboxファイルを追加する。
```
$ vagrant box add centos/7 
```
- インストール中に次のような表示が出てくる。
    - 今回は、virtualboxを使用するので、`3) virtualbox`を選択
```
This box can work with multiple providers! The providers that it
can work with are listed below. Please review the list and choose
the provider you will be working with.

1) hyperv
2) libvirt
3) virtualbox
4) vmware_desktop

Enter your choice: 3
```
- インストールが終わったら、boxがちゃんと作成されたか確認する。
```
$ vagrant box list        
centos/7 (virtualbox, 2004.01)
```
- Vagrantを初期化する。
    - 初期化することで、`Vagrantfile`が作成される。
```
$ vagrant init
```
- Vagrantfileが作成されているか確認しつつ、編集する。
```
$ vi Vagrantfile
```
``` shell:Vagrantfile
  config.vm.box = "centos/7"
```
- Vagrantを起動する。
```
$ vagrant up
```
### 2.3 vagrant sshしてログイン
- Vagrantを起動したので、sshでログインする。
```
$ vagrant ssh
```
### 2.4 yum updateでパッケージをupdateする
```
$ sudo yum update 
```
```
$ sudo vi /etc/yum.repos.d/nginx.repo
```

### 2.5 nginxをインストール
- nginxをインストール。
```
$ sudo yum install -y nginx
```
- firewalld,selinuxを無効化して、192.168.33.15にアクセスできるように設定する。
    - プラグインをインストールする。
```
$ vagrant plugin install vagrant-vbguest
```
- `Vagrantfile`でIP設定されている。まずは、デフォルトの設定を確認する。
```shell:Vagrantfile
  #変更前
  config.vm.network "private_network", ip:"192.168.33.10"
```
- `Vagrantfile`を編集する。
```shell:Vagrantfile
  #変更後
  config.vm.network "private_network", ip: "192.168.33.15"
  config.vm.synced_folder ".", "/var/www/html/", owner: 'vagrant', group: 'vagrant'
```
- `Vagrantfile`を変更したので、リロードする。
```
  $ vagrant reload
```
- ブラウザで、`192.168.33.15`にアクセスすると、次のような画面が表示される。
- これで無事、Nginxがインストールできた！
![welcome_to_nginx.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1159639/cfce2d9c-0a6f-95ad-9a57-ebe5a62cbd09.png)

## 3.参考
- [【備忘録】Vagrant を使用したCentOS7 環境 構築手順](https://qiita.com/Haruka-Ogawa/items/46f784e7f7cd44c6092d)
- [初心者向けシェルスクリプトの基本コマンドの紹介](https://qiita.com/zayarwinttun/items/0dae4cb66d8f4bd2a337)
- [Vagrant環境構築(mac)](https://wiki.adachin.me/archives/360)
- [共有フォルダの設定](https://wiki.adachin.me/archives/235/)
- [virtualboxのIPアドレスを変更する](https://qiita.com/K-N/items/a597b684562e5b1da6a5)
- [Vagrantでnginx](https://ideal-reality.com/computer/server/vagrant-nginx/)
- [Centos7 ファイアーウォール起動/停止、SELinux無効化](https://www.undercoverlog.com/entry/2018/05/02/095248)
