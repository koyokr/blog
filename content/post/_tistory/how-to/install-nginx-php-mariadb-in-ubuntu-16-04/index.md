---
title: Ubuntu 16.04에서 Nginx, PHP, MariaDB 설치하기
slug: install-nginx-php-mariadb-in-ubuntu-16-04
date: 2016-09-13 16:33:00 +0900 KST
categories: [how-to]
markup: mmark
---

나중에 보려고 쓰는 글. 아래 링크 보고 많이 따라했음

<https://blog.lael.be/post/2600>

add-apt-repository 명령어를 쓰기 위해 software-properties-common을 설치

apt install software-properties-common

nginx 저장소 추가 (안정된 버전을 쓰고 싶으면 development 말고 stable로)

add-apt-repository ppa:nginx/development

php 저장소 추가

add-apt-repository ppa:ondrej/php

mariadb 저장소 추가. <https://downloads.mariadb.org/mariadb/repositories/> 에서 직접 보고 하는 걸 추천

```sh
apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xF1656F24C74CD1D8
add-apt-repository 'deb [arch=amd64,i386,ppc64el] http://ftp.kaist.ac.kr/mariadb/repo/10.1/ubuntu xenial main'
```

저장소 추가가 끝났으면 설치

```sh
apt-get update
apt-get install nginx
apt-get install php7.0-fpm
apt-get install mariadb-server
```

처음에 달았던 링크 글에서 널리 쓰이신다고 하신 모듈 설치랑, php와 sql간 연동을 위한 설치

```sh
apt-get install php7.0-gd php7.0-curl php7.0-mbstring
apt-get install php7.0-mysql
```

php.ini에서 date.timezone을 Asia/Seoul로 변경

```sh
vi /etc/php/7.0/fpm/php.ini
vi /etc/php/7.0/cli/php.ini
service php7.0-fpm restart
```

```ini
[Date]
; Defines the default timezone used by the data functions
; http://php.net/date.timezone
date.timezone = Asia/Seoul
```

이렇게

mariadb.cnf 수정

```sh
vi /etc/mysql/conf.d/mariadb.cnf
service mysql restart
```

```ini
# MariaDB-specific config file.
# Read by /etc/mysql/my.cnf

[client]
# Default is Latin1, if you need UTF-8 set this (also in server section)
default-character-set = utf8mb4

[mysqld]
#
# * Character sets
#
# Default is Latin1, if you need UTF-8 set all this (also in server section)
#
character-set-server = utf8mb4
collation-server = utf8mb4_general_ci
character_set_server = utf8mb4
collation_server = utf8mb4_general_ci
```

이건 이렇게

nginx 설정은 기본 설정에서 주석만 떼는 식으로 했다.

index에 index.php를 맨앞에 추가해줬고,

fastcgi_pass unix:/var/run/php7.0-fpm.sock;으로 써진 건 fastcgi_pass unix:/run/php/php7.0-fpm.sock;로 고쳤다.

```sh
vi /etc/nginx/sites-enabled/default
service nginx restart
```

```nginx
root /var/www/html

# Add index.php to the list if you are using PHP
index index.php index.html index.htm index.nginx-debian.html;

server_name _;

location / {
    # First attempt to serve request as file, then
    # as directory, then fall back to displaying a 404.
    try_files $uri $uri/ =404;
}

# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
#
location ~ \.php$ {
    include snippets/fastcgi-php.conf;
#
#   # With php7.0-cgi alone:
#   fastcgi_pass 127.0.0.1:9000;
#   # With php7.0-fpm:
    fastcgi_pass unix:/run/php/php7.0-fpm.sock;
}
```

이렇게 이렇게

root가 /var/www/html로 되어있는데, 난 여기에 index.php를 아래 코드로 생성해서 테스트했다.

80번 포트로 들어가서 페이지가 잘 뜨기를 기도해보자. 안되면 검색ㄱㄱ

```php
<?php
phpinfo();
?>
```
