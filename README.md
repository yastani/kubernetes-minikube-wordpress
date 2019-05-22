# kubernetes-lemp-example
LEMP構成でKubernetesを開発環境に構築する

## LEMPとは

Linux, (E)nginx, MySQL, (PHP|Python|Perl)構成とのこと。

https://www.linode.com/docs/web-servers/lemp/

## 構築手順

```
$ docker login
$ docker build -t django .
$ docker run -itd -p 127.0.0.1:8000:8000 -v src:/code --name yastani django
$ docker exec yastani django-admin startproject myproject .
$ docker exec yastani python3 manage.py startapp todo
```

## .dockerignoreについて

https://qiita.com/munisystem/items/b0f08b28e8cc26132212