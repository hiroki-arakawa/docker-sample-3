# Docker Sample 3

nginx + unicorn Rails + MySQLをDocker Composeで構成するサンプル
モジュールの永続化、DBの永続化、Linkの仕組み？

## How To

```
$ git clone https://github.com/KeitaMoromizato/docker-sample-3
$ cd docker-sample-3
$ sudo docker-compose build
```

Railsアプリを動かすための初期設定

```
$ sudo docker-compose run --rm app bundle install
$ sudo docker-compose run --rm app rake db:create
$ sudo docker-compose run --rm app rake db:migrate
```

`localhost/task`にアクセス

## 解説
### コンテナ上で実行するプロセス

Dockerコンテナは、1つのプロセスのみ実行する。フォアグラウンドでプロセスが動いている間だけコンテナは動くので、バックグラウンドで動かすことはできない。そこで、unicornの起動を`foreman`で管理している。

コンテナ上で実行するコマンドは`docker-compose.yml`のcommandディレクティブで指定できる。

```yml
    command: foreman start -f Procfile.dev
```

### Dockerボリュームの使い方

MySQLのデータをDockerボリュームに保存することで永続化している。コンテナを再起動しても、データが保存される。

`mysql-data`というボリュームを作成し、MySQLのデータが生成されるディレクトリ（`/var/lib/mysql`）にマウントする。

```yml
  mysql:
    image: mysql:5.7.10
    volumes:
      - mysql-data:/var/lib/mysql
volumes:
  mysql-data:
    driver: local
```

### アプリケーションコマンドの実行

アプリケーションを開発していると、データベースのマイグレートやモジュールのインストールコマンドを実行する必要が出てくる。それらのコマンドをホスト上で実行すると、コンテナとの環境差異によってエラーが出る可能性がある。よって、コマンドはコンテナ上で実行する。

```
$ sudo docker-compose run --rm <コンテナ名> <コマンド>
```

コマンド実行後にコンテナを削除するときは`---rm`オプションをつける。

### モジュールのインストール

RubyのGem、node.jsのnpmのようにアプリケーションを開発するためには、モジュールをインストールし、活用するのが一般的。それらのモジュールも、永続化しないとコンテナを起動するたびにインストールが必要になる。

よく`Dockerfile`内でインストールを完了しているサンプルを目にするが、開発初期は頻繁にモジュールのインストールが発生するので、その度にDockerのビルドをするのは効率が悪い。

解決策としては、以下の2つが考えられる。

1. gemのインストール先ディレクトリをローカルにマウントして永続化する
2. コンテナにvolumeをマウントし、volume上にgemをインストールするように設定する

どちらでもよいが、エディタ上でeslintを使っている場合など、ホスト上にモジュールが存在するかをチェックするケースがあるので、今回は1を選択する。

`Dockerfile`上でGemのインストール先を`vendor/bundle`に変更している。

```Dockerfile
RUN \
  bundle config --local path vendor/bundle && \
```
