# docker-wordpress-nginx


This is a fork of eugeneware's Dockerfile for Wordpress.

My plan is to allow the use of linked containers for Mysql and data volumes.

Currently you can link to another Mysql container.


## Installation


If you'd like to build the image yourself then:

```bash
$ git clone https://github.com/eugeneware/docker-wordpress-nginx.git
$ cd docker-wordpress-nginx
$ docker build -t="rija/docker-wordpress-nginx" .
```

## Usage

To spawn a new instance of wordpress on port 80.  The -p 80:80 maps the internal docker port 80 to the outside port 80 of the host machine.

```bash
$ docker create --name wp-content -v /usr/share/nginx/www/wp-content docker-wordpress-nginx
$ docker run --name wordpress-server --volumes-from wp-content -d -p 80:80 --link mysql-server:db docker-wordpress-nginx
```

You will need to have started a Mysql container before hand:

```bash
$ docker create --name mysql-data -v /var/lib/mysql mysql:5.5.42
$ docker run --name mysql --volumes-from mysql-data -e MYSQL_ROOT_PASSWORD=<root password> -e MYSQL_DATABASE=wordpress -e MYSQL_USER=<user name> -e MYSQL_PASSWORD=<user password> -d mysql:5.5.42
```


After starting the docker-wordpress-nginx check to see if it started and the port mapping is correct.  This will also report the port mapping between the docker container and the host machine.

```
$ docker ps

CONTAINER ID        IMAGE                           COMMAND                CREATED             STATUS              PORTS                NAMES
e1afad973636        docker-wordpress-nginx:latest   "/bin/bash /start.sh   25 minutes ago      Up 25 minutes       0.0.0.0:80->80/tcp   wordpress-server    
5ef15c984107        mysql:5.5.42                    "/entrypoint.sh mysq   37 minutes ago      Up 37 minutes       3306/tcp             mysql-server        

```

You can then visit the following URL in a browser on your host machine to get started:

```
http://127.0.0.1:80
```

If you want to backup the content of Wordpress' wp-content folder you can use the following command.

```bash
$ docker run --rm --volumes-from wp-content -v $(pwd):/backup docker-wordpress-nginx tar cvf /backup/backup.tar /usr/share/nginx/www/wp-content
```
