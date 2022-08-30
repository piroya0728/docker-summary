# docker コマンド集


## 1. Dockerの仕組み

### Dockerの基本構造について

#### Docker Engine

Dockerを利用するための常駐プログラム

Docker for mac、Docker for Windows、Docker ToolboxなどのソフトウェアをPCにInstallすることで、常駐プログラムとしてDocker Engineが動作し、Dockerを利用することができるようになります。

2. Image
Dockerのimageにはtagという概念があります。
tagとは「imageのversion」のことです。
image名とタグ名を : で区切って

```
nginx:latest  
nginx:1.14-perl
```
のように表されます。

#### Imageの構造
1．　layer構造になっている
一度作成されたimageは編集不可能 (読み取り専用)

Imageを用意する

ローカルマシンに取得しているImage一覧は
```
$ docker images
```
コンテナを実行するには、そのコンテナの元となるimageを用意しなくてはなりません。

imageを用意する方法には

1. 他人が作ったimageを取得する(主にDocker Hubから)
2. 自分で作る(他人のimageを改良する)  
があります。

1. Docker Hubからのimage取得
Docker Hubには様々なミドルウェアが入ったImageが公開されているので、自分が使用したい環境に応じて以下のコマンドでImageを指定して取得します。
```
$ docker pull イメージ名     #Docker Hubなどからイメージを取得
```
2. 2. Dockerfileからのイメージ作成
Dockerfiile という名前のtextファイルを作り、その中にimage build(image作成)に関わる設定を記述して、そのDockerfileからイメージを自作します。

様々な記述ができますが、ここでは基礎的な以下の関数のみを紹介します。

```
FROM イメージ名:タグ名　　　　　　　#どのimageを元にimageを作成するか  
  
RUN パッケージ等インストールコマンド     #ここに記述したコマンドを実行してミドルウェアをインストールし、imageのレイヤーを重ねる  
  
CMD コマンド指定　　　　　　　　#コンテナが作成された後で実行するコマンドを指定する
```

-tオプションで作成したimageに名前をつけます。

また、image作成する際にdocker daemonに転送されるファイルの範囲をビルドコンテキストといい、イメージ名の後に指定します。(今回は . [カレントディレクトリ])

buildコマンド実行時に設定として読み込むDockerfileはデフォルトではビルドコンテキスト上のDockerfiileが読み込まれるようになっているので、ビルドコンテキストにDockerfileを置いておく必要があります。
※ 別途-f Dockerfile名で指定することもできます。

一度buildしたimageを、Dockerfileの変更無しに再びbuildする際にはbuild cacheが効いて、以前buildした情報を見に行くようになっているので、RUNにupdateコマンドなどを記述していたとしても普通にbuildし直した場合は、そのコマンドは実行されないことになる点に注意が必要です。

cacheを無効にしてbuildする場合は
```
$ docker build —no-cache -t docker-whale
```


## 3. コンテナ
imageを用意したら、そのイメージからコンテナ(=アプリケーションの実行環境)が起動できます。

### imageからコンテナの起動
```
$ docker create --name コンテナに付ける名前 イメージ名
```

でイメージからコンテナを作成

```
$ docker start コンテナ名
```
でコンテナを起動

これまで見てきた指定したイメージをDockerHubから取得して、コンテナを作成・起動するコマンドとして

```
$ docker run イメージ名
```
があります。

docker pull :DockerHubからイメージを取得
docker create ：取得したイメージからコンテナを作成
docker start ：作成したコンテナを起動

を同時に実行するコマンドがdocker run コマンドです。

### コンテナの操作コマンド

#### コンテナ一覧を確認
```
$ docker ps
```
で現在実行中のコンテナを表示
```
$ docker ps -a
```
で現在存在しているコンテナ一覧を表示

#### コンテナの詳細を確認
```
$ docker inspect コンテナ名
```
で指定したコンテナの詳細情報を表示

#### コンテナの作成
```
$ docker create —name コンテナにつける名前 -it イメージ名 /bin/bash
-i はコンテナの標準入力を取得して双方向接続するオプション
-t はコンテナ内にTTYを割り当てるオプション
```

コンテナでシェル実行をして、フォアグラウンドで実行状態にしておきたい場合に -itの組み合わせで使われる
これをつけないとシェルがすぐに終了してコンテナが停止してしまう

#### コンテナの起動
```
$ docker start コンテナ名
```

でコンテナを起動

#### コンテナの一時停止
```
$ docker pause コンテナ名
```
で、コンテナを一時停止できる

#### コンテナの再開
```
$ docker unpause コンテナ名
```
でコンテナの一時停止を解除できる

#### コンテナの再起動
```
$ docker restart コンテナ名
```
でコンテナが再起動される

#### コンテナの停止
```
$ docker stop コンテナ名
```
でコンテナが停止

コンテナが終了後、そのままローカルに残っている状態はExitとなることに注意が必要です。
使わないコンテナは削除しておかないとハードディスクが圧迫されます。

#### コンテナの削除
```
$ docker rm コンテナ名
```
でコンテナを削除
```
$ docker rm -f コンテナ名
```
で強制削除

#### 起動中のコンテナのシェルへ接続
接続方法はattachとexecの2つがあります。
```
$ docker attach 起動中コンテナ名
```
起動中のコンテナがシェルを実行している場合は、シェルに接続できるが、Deamonしか起動していない場合にはDeamonの標準入出力に接続されてしまうことに注意して下さい。

attachで接続して、exitで抜けると起動中のコンテナが終了してしまうのが特徴です。
起動時に -itフラグをつけている場合は、control + p, control + qで抜けるとコンテナは終了しません。
```
$ docker exec -it 起動中のコンテナ名 /bin/bash
```
exitで抜けることができ、抜けてもコンテナが停止することはないのが特徴です。

コンテナからイメージを作成
```
$ docker commit コンテナ名 image名：タグ名
```
コンテナの状態を、指定した「image名：タグ名」でイメージとして保存するコマンド
```
$ docker history イメージ名
```
で、そのイメージの変更履歴を確認できますが、docker commitでイメージを作成すると、コンテナの内部でどんな変更を加えたのかはどこにも記録として残らず、とても使いにくいimageになってしまう点に注意が必要です。

そのため、基本的にはDockerfileからimageに変更を加えてイメージ自作をするべきです。


## 4. Docker Hub
### Docker Hubでのimage共有
Githubと殆ど同じです。
Dockerhubのアカウントを持っている必要があります。

手順としては

1. Docker hub上でリポジトリを作成する

2. ローカルでimageを作成

3. ローカルのコマンドラインで以下のコマンドを使ってdocker hubへログイン
```
$ docker login
```
4. 以下のコマンドでpushしたいimageにタグ付け
```
$ docker tag <pushしたいimage名> <docker hub ID>/<新規image名>:<タグ名>
```
5. 以下のコマンドでローカルのimageをリモートリポジトリにpushする
```
$ docker push ＜docker hub ID＞ / ＜image名＞：＜タグ名＞
```

## 5. Dockerにおけるデータ管理

起動したコンテナ内で扱う動的なデータは、読み書き可能な最上レイヤー(コンテナレイヤー)に置くこともできますが、

- コンテナが削除された時点でそのコンテナ内のデータは消える
- ・コンテナ間でデータ共有できない
・コンテナレイヤーへのデータ書き込みは、通常のファイルシステムと異なるユニオンファイルシステムが使われているため、書き込み速度が遅い

というデメリットがあります。

そのため、Dockerではホストマシン上にデータを管理し、それをコンテナにマウントする手法が使われます。

手法は主に3つあり、以下で説明して行きます。

### volume
ホストマシン上に自動生成される指定ディレクトリ(/var/lib/docker/volumes)をコンテナにマウントする手法

ホストマシン上で
```
$ docker volume create volumeの名前
```
とすることで、volume(/var/lib/docker/volumesディレクトリ)を作成し、コンテナを起動する際に
```
$ docker run -itd --name 作成するコンテナ名 --mount source=[マウントするvolume名],target=[コンテナ上のマウント先ディレクトリ] イメージ名

具体例：
$ docker run -itd --name mount-test --mount source=volume1,target=/app nginx
```
のように、--mountオプション(-vオプションでも可)をつけることで指定したvolumeをマウントすることができます。

マウントしたvolumesディレクトリは、マウント元であるホスト上のディレクトリからは直接操作するべきではない点に注意が必要です。

また、同じホスト内で立てている異なるコンテナでも、それぞれ同じvolumeをマウントすることでファイルの共有ができます。

さらに、volumeを複数のコンテナで共有させる場合には、コンテナごとに編集権限を設定することもできます。
```
$ docker run -itd —name mount-c4 —mount source=copy-vol, destination=/etc/nginx,readonly nginx
```

のように , で区切って編集権限を指定します。

### volumeの管理コマンド
#### volumeの一覧を確認
```
$ docker volume ls
```
volumeの詳細を確認
```
$ docker volume inspect volume名
```
volume を削除
```
$ docker volume rm volume名
```
コンテナを削除してもvolumeは残り続けるので、$ docker volume rm volume名　で削除する必要がある点に注意が必要です。

### bind mount
bind mount は、ホストマシン上の任意のディレクトリをマウントでき、ホスト側のディレクトリを直接操作をしても良いという点がvolumeと異なるマウント手法です。

volumeのように事前に設定する必要はなく、以下のコマンドでコンテナの起動時にオプションで指定してマウントします。
```
$ docker run -itd --name [コンテナ名] --mount type=bind,source=[マウント元ディレクトリ],target=[マウント先ディレクトリ] イメージ名

具体例:
$ docker run -itd —-name bind-mount-test —-mount type=bind,source=“$(pwd)”/mount,target=/app nginx
```
source(マウント元)のディレクトリが存在しない場合は、エラーになるので、事前に作成しておきましょう。(-vオプションを使用した場合は自動作成される)

bind mountでは、もしホスト上の空のディレクトリをコンテナ上の/userなどにマウントしたりした場合は、コンテナ上のデータが消えてしまい、コンテナがまともに動作しなくなることもあるので注意が必要です。

volume の時と同様の方法で、コンテナごとに編集権限を設定することもできます。

### tmpfs(テンプfs)
ホストマシンのメモリ領域をコンテナ上にマウントする手法

ホストマシンが終了した場合も、コンテナが終了した場合も、保持していたデータは解放されます。
```
$ docker run -itd --name [コンテナ名] --mount type=tmpfs,destination=[マウント先ディレクトリ] イメージ名

具体例:
$ docker run -itd --name tmpfs-test --mount type=tmpfs,destination=/app nginx
```
のようにコンテナ起動時に、--mountオプションのtypeにtmpfsを指定することでマウントができます。

また、ホスト上のメモリを無制限に使用してしまう可能性を排除するために、
```
$ docker run -itd --name tmpfs-test --mount type=tmpfs,destination=/app,tmpfs-size=800000000,tmpfs-mode=800 nginx
```
のようにオプションで使用可能なメモリサイズを制限することもできます。

## 6. Docker Compose
Docker Composeは複数コンテナのDocker アプリケーションを事前定義して実行するためのツールです。

コンテナ1つを起動するにも、volumeなどのマウント設定や所属させるDockerネットワークの設定などを明記して
```
$ docker run -itd —name mount-test —mount source=volume-test, destination=/etc/nginx,readonly nginx
```
のような複雑なオプション付きのコマンドを打ち込まなければなりません。

これはとても面倒ですし、ミスにも繋がり、円滑に環境の共有が出来ない原因になりかねません。

そこで、それらを自動化できるのがDocker Composeです。

あるWebサービスの実行環境をDockerで構築する場合、Webサーバー, DBサーバー, Cacheサーバーなどの定義を一つのdocker-compose.ymlファイルに記述しておくことによって、それを元に実行に必要なコンテナをまとめて起動・設定することができます。

手順は以下の通りです。

1. Dockerfileを用意する または Docker Hubなどに使用するイメージを用意する
2. docker-compose.ymlを定義する
3. ymlファイルがあるディレクトリで、$ docker-compose upを実行する

### docker-compose管理コマンド一覧

docker-composeで起動したコンテナの一覧を表示
```
$ docker-compose ps
```
docker-compose.ymlに記述した設定からコンテナ起動
```
$ docker-compose up
```
既にコンテナが起動中であっても再起動してくれる

docker-composeで作成されたコンテナやネットワークを削除
```
$ docker-compose down
# $ docker-compose down -v
```
-vオプションでvolumeも一緒に削除される

指定したサービスコンテナ内でコマンドを実行
```
$ docker-compose run [コンテナ名]　 [コマンド]
```
一連のコンテナ停止
```
$ docker-compose stop
```
一連のコンテナ起動
```
$ docker-compose start
```
### docker-composeによるアプリ実行環境構築
PythonのWAFであるDjangoを使用したWebアプリの環境構築

1. imageの用意
作業ディレクトリを作成してその中にDockerfileを作成します。
```
Dockerfile
FROM python:3  # image名にpython3実行環境のイメージを指定
ENV PYTHONUNBUFFERED 1 # pythonの標準出力をバッファにため込まないための環境変数設定
RUN mkdir /service  # serviceディレクトリを作成
WORKDIR /service # serviceディレクトリに移動
COPY requirements.txt /service/  # requirements.txt(事前に作成)をserviceディレクトリに置く
RUN pip install -r requirements.txt # pip  installでパッケージをインストール
COPY . /service/ # buildコンテキストの内容を全て/service内に置く
```
#### docker-compose.ymlの作成
以下のようにdocker-compose.ymlファイルを作成します。
```
docker-compose.yml
version: '3' # docker-composeのversion指定
services: # 起動するサービスコンテナを書いていく
  db: # dbサーバーコンテナを起動
   image: postgres # イメージはdockerhub上のpostgres:latestを使用
  web: # webサーバーコンテナを起動
   build: . # カレントディレクトリにあるdockerfileからimage作成
   command: python3 manage.py runserver 0.0.0.0:8000  ＃ コンテナ起動時に実行されるコマンドを指定
   volumes:
    - .:/app # currentディレクトリを/appディレクトリにbind mountする
   ports:
    - "8000:8000" # コンテナの8000番を公開
   depends_on: # webサーバーコンテナを立ち上げる前にdbサーバーコンテナを立ち上げるようにする
    - db
```
depends_onは立ち上げコマンドを実行する順番を規定するだけで、「dbが起動完了してからwebを実行する」訳ではないです。
このことが原因でdocker-compose upが上手く行かない場合があるので、その場合はentrypoint:で起動待ちをするshell scriptを実行するなどして対処します。

http://docs.docker.jp/compose/startup-order.html

3. docker-composeの実行
```
$ docker-compose run web 
```
django-admin.py startproject test .
でdocker-compose.ymlに定義したwebサービスコンテナを一時起動し、djangoプロジェクトを作成します。
```
$ docker-compose up -d
```
でデタッチドモード(-d : バックグラウンド)で一連のコンテナを起動します。

このdocker-compose.ymlファイルを共有することで、別のマシンでも $ docker-compose up コマンド一つで一連のコンテナを実行することができます。