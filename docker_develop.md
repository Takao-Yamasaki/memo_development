# 米国AI開発者がゼロから教えるDocker講座

## Dockerの動きをもう少し詳細に理解する

### $ dokcer.runは何をしているのか？
- (1)ホスト（ローカル）のDockerイメージからコンテナを作成
- (2)デフォルトコマンドを実行する
- つまり、run = create + start
```
$ docker run hello-world
Hello from Docker!
```
- 実行されると、もともと文字が表示されるだけなので、すぐExited状態になる。
```
docker ps -a
CONTAINER ID   IMAGE                                     COMMAND                  CREATED         STATUS                     PORTS                               NAMES
3211f49d8bcf   hello-world                               "/hello"                 3 minutes ago   Exited (0) 3 minutes ago                                       confident_moser
```
- 試しに、createを実行してみる
```
$ docker create hello-world
```
- created状態になる
```
$ docker ps -a
CONTAINER ID   IMAGE                                     COMMAND                  CREATED          STATUS                     PORTS                               NAMES
214d205778b2   hello-world                               "/hello"                 16 seconds ago   Created                                                        youthful_zhukovsky
```
- デフォルトコマンドを実行する。
```
$ docker start 214d205778b2
```
- たしかに、デフォルトコマンドは実行されているはずなのに、Exited状態になっている。
- ちなみに、デフォルトコマンドというのは、`"/hello"`の部分。
```
$ docker ps -a
CONTAINER ID   IMAGE                                     COMMAND                  CREATED          STATUS                      PORTS                               NAMES
214d205778b2   hello-world                               "/hello"                 7 minutes ago    Exited (0) 3 minutes ago                                        youthful_zhukovsky
```
- `-a`をつけると実行結果が表示される
```
$ docker start -a 214d205778b2
Hello from Docker!
```

### コマンドの上書き
- デフォルトコマンドは、`"/hello"`だが、`bash`で上書きしている
```
$ docker run -it hello-world bash
docker: Error response from daemon: OCI runtime create failed: container_linux.go:380: starting container process caused: exec: "bash": executable file not found in $PATH: unknown.
ERRO[0000] error waiting for container: context canceled
```

- 以下のコマンドだと、コンテナが作られて（create）、デフォルトコマンドを実行(start)して、Exited状態となる。
```
$ docker run ubuntu
```

- ubuntuの場合、bashで上書きしなくても、デフォルトコマンドが"bash"になっている。
```
$ docker ps -a
CONTAINER ID   IMAGE                                     COMMAND                  CREATED          STATUS                      PORTS                               NAMES
565b318b58a0   ubuntu                                    "bash"                   24 seconds ago   Exited (0) 23 seconds ago                                       stoic_kirch
```
`ls`で上書きすると、コマンドが実行された。
```
docker run ubuntu ls
bin
boot
dev
etc
home
lib
lib32
lib64
libx32
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
```

### -itって何しているの？
- -i インプット可能
  - ホストからコンテナへの入力チャネル（STDIN）を開く
  - インタラクティブ（対話的）な操作を可能にする
- -t 綺麗にする
- bashを使うときは、インタラクティブな操作をしたいので。`-it`をセットで使う

### コンテナの削除
- UP状態になっているコンテナを削除してみる
```
$ docker ps -a
CONTAINER ID   IMAGE                                     COMMAND                  CREATED             STATUS                         PORTS                               NAMES
cb9fe3b2a55a   ubuntu                                    "bash"                   40 hours ago        Up 39 hours                                                        dazzling_bell
```

- 当然エラーとなる
```
$ docker rm cb9fe3b2a55a
Error response from daemon: You cannot remove a running container cb9fe3b2a55a137f4444bac19bbfc6f71809347bc43de64e97a695a962eb88b1. Stop the container before attempting removal or force remove
``` 

- 一旦ストップして、Exited状態として、削除する
```
$ docker stop cb9fe3b2a55a
$ docker rm cb9fe3b2a55a
```

- UP以外のコンテナを全部削除する(めっちゃ便利)
```
$ docker system prune
WARNING! This will remove:
  - all stopped containers
  - all networks not used by at least one container
  - all dangling images
  - all dangling build cache

Are you sure you want to continue? [y/N] y
```

### ファイルシステムの独立性
- 同じイメージから作成したコンテナでも、それぞれファイルシステムは独立している。

- 複数コンテナを停止する
```
$ docker stop 8974f5c27b28 dc65e3496e2d
```
- 複数コンテナを削除する
```
$ docker rm 8974f5c27b28 dc65e3496e2d
```

### コンテナ名を指定してrunする
- コンテナを起動し続ける時は、nameを指定したほうが便利
```
$ docker run --name sample_container ubuntu
```
ステータスを確認
```
$ docker ps -a
CONTAINER ID   IMAGE                                     COMMAND                  CREATED          STATUS                      PORTS                               NAMES
00a5c8a66a8d   ubuntu                                    "bash"                   27 seconds ago   Exited (0) 25 seconds ago                                       sample_container
```

### detachedモードとforegroundモード
- detachedモードは、バックグラウンドでコンテナを起動したままにするモード。
- バックグラウンドで常に起動させたいものに使用する。
- `-d`のオプションをつける。
```
$ docker run -d -it ubuntu bash
```
- デタッチしているので、ステータスはUPのままの状態
```
$ docker ps -a
CONTAINER ID   IMAGE                                     COMMAND                  CREATED          STATUS                      PORTS                               NAMES
a2ca611a6d4b   ubuntu                                    "bash"                   32 seconds ago   Up 31 seconds                                                   frosty_ishizaka
```

- foregroundモードは、通常のモード。
- `--rm`をつけると、Exitした後にすぐコンテナを削除する。
- 一回きりの使い捨てコンテナに使用する。
```
$ docker run --rm hello-world

Hello from Docker!
```

## Dockerfileとは？
- Dockerイメージを作る元となるテキストファイル（設計図）のようなもの。
- `INSTRACTION arguments`の形で書く。
- INSTRACTION:コマンドのようなもの
- arguments: 引数

### Dockerfileを作ってみよう！
- Dockerfileを作成
```Dockerfile
# 元となるDockerイメージを指定する
FROM ubuntu:latest

# testファイルを作成
RUN touch test
```
- DockerfileからDockerイメージを作成する
```
$ docker build .
```

- イメージを確認
- noneの状態のことをダングリングイメージという
```
$ docker images
REPOSITORY                            TAG       IMAGE ID       CREATED          SIZE
<none>                                <none>    bb09618b29d9   27 seconds ago   77.8MB
```

- このようにフィルターをかけて、ダングリングイメージだけ抽出することができる
```
$ docker images -f dangling=true
REPOSITORY   TAG       IMAGE ID       CREATED              SIZE
<none>       <none>    bb09618b29d9   About a minute ago   77.8MB
```

- `ーt`で名前とタグを付ける
```
docker build -t new-ubuntu:latest .
```

- イメージを確認する
```
$ docker images
REPOSITORY                            TAG       IMAGE ID       CREATED         SIZE
new-ubuntu                            latest    bb09618b29d9   4 minutes ago   77.8MB
```

### ビルドしたDockerイメージをrunする
- new-ubuntu（OS）のbashを実行する
```
$ docker run -it new-ubuntu bash
```
- 基本的には、Dockerfileをメンテして、作業内容を可視化する

## Dockerfileの書き方の基本
### FROM
- ベースとなるイメージを作成する
- DockerfileはFROMから始める
- Linuxのコマンドと打つためには、Linux系のOSがまず必要なので、ubuntuをまず入れている

### RUN
- Linuxのコマンドを実行するもの
- RUN毎にイメージレイヤーを作成している


### Layer数を最小限にするために
- RUN / COPY / ADDでLayerを作る
  - 必要なパッケージをインストールすることが一番多い
  - `apt-get`を使ってパッケージ管理をする
- コマンドを&&で繋げる
```
$ touch test2 && echo 'Hello world' > test2
```
- バックスラッシュで改行する（アルファベット順に並べる）

### cacheを使おう
- パッケージのインストールには時間がかかる
- Dockerfile作成中は、キャッシュをうまく使いたいので、複数のRUNを書く
- 最後に&&を使って、コードを整理してベストプラクティスに近づけていく

```Dockerfile
# 元となるDockerイメージを指定する
FROM ubuntu:latest

# 一覧を取得して、パッケージをインストール
# -yで全てyesで答えるようにする
RUN apt-get update
RUN apt-get install -y \
    curl \
    nginx
RUN apt-get install -y cvs
```

### CMD
- デフォルトのコマンドを指定するもの
- 原則として、Dockerfileの最後に記述する
  - 1つのDockerfileに1つだけ記述する
```Dockerfile
# 元となるDockerイメージを指定する
FROM ubuntu:latest

# 一覧を取得して、パッケージをインストール
# -yで全てyesで答えるようにする
RUN apt-get update && apt-get install -y \
    curl \
    cvs \
    nginx
CMD [ "pwd" ]
```
- Dockerイメージを作成
```
$ docker build .
```
- コンテナを作成し、デフォルトコマンドを実行
  - ケースバイケースだが、`bin/bash`を指定するケースが多いかも。
```
$ docker run 5a7e040a8b534b8e5c6
```

### RUN vs CMD
- RUNは、Layerを作成する（Dockerイメージに持たせたい）
- CMDは、作らない

## `docker build .`の詳細
### Dockerの構成
- Dockerは、クライアントサーバーアーキテクチャを採用している
- ユーザーは、Docker CLIを使って、Dockerデーモンに対して指示出しして、それがコンテナやイメージに対して命令している
- クライアントサーバー方式だけど、実際は、ホスト上でしか動いていないことが多いため、意識することは少ない

- クライアント
  - docker run
  - docker images
  - docker pull
- Dockerホスト
  - Docker Deamon(デーモン) 
    - コンテナ
    - イメージ

### build contextとは？
- `docker build .`の時に指定したフォルダを`build context`(「ビルドをするときの環境」という意味)と呼ぶ
- `docker build .`を実行すると、ホストからbuild contextがDockerデーモンに渡されて、build context内にあるファイルを使ってイメージをビルドする
  - そのため、ビルドに使用しないファイルが`build context`内に置かないのが鉄則

- データサイズがわかる
  - du: data useageの略
```
$ du -sh <filename>
```
- ADDやCOPYをすると、build contextの中にあるファイルをimageに持っていける

### COPY
- COPYを使って、ホストにあるファイルをコンテナ内にコピーをする
```Dockerfile
# 元となるDockerイメージを指定する
FROM ubuntu:latest
RUN  mkdir /new_dir
COPY something /new_dir
```
- イメージを作成する
```
$ docker build .
```
- コンテナを起動して、bashを実行する。
```
docker run -it 9f0678b9e4c714 bash
```

### COPY vs ADD
- COPY 単純にファイルやフォルダをコピーする
- ADD tarの圧縮ファイルをコピーして解凍するときに使用する。大きいファイルを使うとき。

- サンプルフォルダを作成
```
$ mkdir sample_folder
```
- sample_folderのなかに移動して、helloファイルを作成
  - このようにするとhelloファイルの中にhelloが記述される
```
$ echo 'hello' > hello
```

- sampleフォルダを圧縮する
```
$ tar -cvf compressed.tar sample_folder
```

- イメージを作成
```
$ docker build .
```

- コンテナを作成して、bashを実行し、起動後削除する
```
$ docker run -it --rm 996da1503f2bb663357 bash
```

- 中身を確認すると、フォルダが解凍された状態でコピーされた
```
root@27c6fb429d26:/sample_folder# ls
compressed.tar  hello
```

### Dockerfileがbuild contextにない場合
- Dockerfileを複数作成している場合は、`-f`でパスを指定してあげることが多い
- mvコマンドを使って、リネームする
  - 開発環境のときは`dev`、テスト環境のときは`test`を指定する場合が多い
```Dockerfile
$ mv Dockerfile Dockerfile.dev
```

- `-f`でDockerfileのあるパスを指定する
- `build context`はカレントフォルダになるので、`.`を指定する
```
$ docker build -f ../Dockerfile.dev .
```

### ENTRYPOINT
- ENTRYPOINTも、CMDと同じでデフォルトコマンドを指定できる
- ENTRYPOINTは、RUNの時に上書きできない。
- ENTRYPOINTがある時は、CMDはENTRYPOINTの引数のみを取る
- この場合、上書きは、引数しかできない

```Dockerfile
# 元となるDockerイメージを指定する
FROM ubuntu:latest
RUN touch test
ENTRYPOINT [ "ls" ]
CMD [ "--help" ]
```

### ENV
- 環境変数を設定してくれるインストラクション

- 設定されているパスを確認するコマンド
```
echo $PATH
```

```Dockerfile
FROM ubuntu:latest
ENV key1 value
ENV key2=value
ENV key3="v a l u e" key4=v\ a\ l\ u\ e
ENV key5 v a l u e
ENV name takao
```

- いつものように、イメージを作成し、コンテナを起動し、デフォルトコマンドを実行
```
$ docker build .

$ docker run -it --rm  5e015251577 bash 
```

- 環境変数の一覧を表示する
```
$ env 
key4=v a l u e
key5=v a l u e
key2=value
key3=v a l u e
key1=value
```

### WORKDIR
- Dockerインストラクションの実行フォルダを変更する
  - RUNは、全てルート直下で実行される（たとえcdコマンドを使ったとしても）

- この書き方なら、sample_folder直下にsample_fileが作成される
```Dockerfile
FROM ubuntu:latest
RUN mkdir sample_folder && \
cd sample_folder && \
touch sample_file
```

- この書き方をすると、それ以降のインストラクションについては、`/sample_folder`配下で実行され、ファイル配下に移動する
  - その歳、`WORKDIR <絶対パス>`で指定すること
```Dockerfile
# 元となるDockerイメージを指定する
FROM ubuntu:latest
WORKDIR /sample_folder
RUN touch sample_file
```

## ホストとコンテナの関係
- ファイルシステムの共有
- ファイルへのアクセス権限
- ポートを繋げる
- コンピュータリソース（CPU メモリー）の上限

### ファイルシステムを共有する
- `-v`を使って、ホストのファイルシステムをコンテナにマウントする
  - ホストにコードを記述したファイルを配置して、そのファイルがあたかもコンテナ上にあるような振る舞いをさせる
  - コードの肥大化を防ぐため
- 実行環境として使用することが一般的
- コンテナを起動する時に、`-v`をつけて実行する
  - hostのパス:containerのパス
```
$ docker run -it -v ~/Documents/mounted_folder:/new_dir b03ae9af bash
```

### ホストとコンテナのアクセス権を共有する
- ユーザーIDの確認
```
$ id -u
501
```

- グループIDの確認
```
$ id -g
20
```

- ホストと同じ権限（ユーザーとグループ）を指定して、runする
```
$ docker run -it -u $(id -u):$(id -g)  -v ~/Documents/mounted_folder:/created_in_run 0256d8c bash
```

- パーミッションの見方
  - d: ディレクトリ l: シムリンク
  - 所有者
  - 所有グループ
  - その他

- rootで作成したフォルダになるので、書き込み権限がない
```
$ ls -la
drwxr-xr-x   2 root root 4096 Jul  9 04:33 created_in_Dockerfile
$ touch test
touch: cannot touch 'test': Permission denied
```

- 501が作成したフォルダになるので、書き込みが可能
```
$ ls -la
drwxr-xr-x   3 root root   96 Jul  9 04:32 created_in_run
$ touch test
```

### ホストとコンテナをポートで繋げる
- hostとcontainerのポートを接続する。publishする
- `-p <host_port> <container_port>`で記載する
  - containerのポートは、jupyterのデフォルトのポート番号なので、基本的には変えない
```
$ docker run -it -p 8888:8888 --rm jupyter/datascience-notebook bash
```

### リソースの上限を設定する
- 自分のPC上であれば、設定する必要はないが、共有サーバーを使用する場合は、指定してあげる必要がある
- 物理コア数の確認
```
$ sysctl -n hw.physicalcpu_max
2
```
- 論理コア数の確認
```
$ sysctl -n hw.logicalcpu_max
4
```

```
$ sysctl hw.memsize
hw.memsize: 8589934592
8589934592 / (1024 *1024 * 1024)
8
```

- cpuを2、memoryを2GBに指定
```
$ docker run -it --rm --cpus 2 --memory 2g ubuntu bash
```

```
$ docker ps
CONTAINER ID   IMAGE                                     COMMAND                  CREATED          STATUS          PORTS                               NAMES
ca6a90fa403b   ubuntu                                    "bash"                   35 seconds ago   Up 35 seconds                                       focused_ganguly
```
- `docker inspect`でコンテナの詳細情報を表示する
  - grepでcpuと記載がある項目を抽出する
  - `-i`は`ignore`で大文字小文字を区別しない
```
$ docker inspect ca6a90fa403b671cb | grep -i cpu
            "CpuShares": 0,
            "NanoCpus": 2000000000,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "CpuCount": 0,
            "CpuPercent": 0,

2000000000 * 10^-9
CPU: 2
```

```
$ docker inspect ca6a90fa403b671cb | grep -i memory
            "Memory": 2147483648,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 4294967296,
            "MemorySwappiness": null,

1K byte = 1024 byte
2147483648 / (1024 * 1024 * 1024)
2GB
```


