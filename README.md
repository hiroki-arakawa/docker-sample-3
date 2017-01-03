# Docker Sample 3

モジュールの永続化、DBの永続化、Linkの仕組み？

nginx + unicorn Rails

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
