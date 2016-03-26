# Typecho with docker

## First create a mysql container

```bash
docker run --name mysql -d -e MYSQL_ROOT_PASSWORD=my-secret-pw -v /place/to/store/mysql:/var/lib/mysql mysql:5.6
```

## Second php-fpm

I have configured a php-fpm for you, just config a volume is ok

```bash
docker run --name phpfpm -d -v /data/web/typecho/wwwroot:/app --link mysql:mysql jimmyzhou/typecho-nginx-php
```

## Third nginx

I have configured rewrite in the provided `nginx.conf`, just open it forcedly 

``` bash
docker run --name typecho_nginx -d -p 80:80 --link phpfpm:phpfpm -v /data/web/typecho/conf/nginx.conf:/etc/nginx/nginx.conf:ro --volumes-from phpfpm nginx:1.9
```

## Download  typecho
```bash
cd /data/web/typecho/wwwroot
wget https://github.com/typecho/typecho/releases/download/v1.0-14.10.10-release/1.0.14.10.10.-release.tar.gz
tar zxf 1.0.14.10.10.-release.tar.gz
mv ./build/* ./
rm -rf 1.0.14.10.10.-release.tar.gz
rm -rf ./build
```

# FAQ 

## if you want to restore data

```bash
docker stop typecho_nginx
docker run --name myadmin -d --link mysql:db -p 80:80 phpmyadmin/phpmyadmin
```

After you restore it, 

```bash 
docker stop myadmin
docker start typecho_nginx
```

## In config.inc.php

When installing, you should configure your database information as phpinfo tell you, but after installed, you should config your database like this
```php
$db = new Typecho_Db('Pdo_Mysql', 'typecho_');
$db->addServer(array (
  'host' => getenv('MYSQL_PORT_3306_TCP_ADDR'),
  'user' => 'root',
  'password' => getenv('MYSQL_ENV_MYSQL_ROOT_PASSWORD'),
  'charset' => 'utf8',
  'port' => getenv('MYSQL_PORT_3306_TCP_PORT'),
  'database' => 'typecho',
), Typecho_Db::READ | Typecho_Db::WRITE);
Typecho_Db::set($db);
```