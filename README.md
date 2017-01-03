# Docker Sample 3

モジュールの永続化、DBの永続化、Linkの仕組み？

nginx + unicorn Rails

## How To



```
$ sudo docker-compose run --rm app bundle install
$ sudo docker-compose run --rm app rake db:create
$ sudo docker-compose run --rm app rake db:migrate
```
