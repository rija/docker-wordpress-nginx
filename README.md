# docker-wordpress-nginx


This is a fork of eugeneware's Dockerfile for Wordpress.

My plan is to allow the use of linked containers for Mysql and data volumes.

Currently you can link to another Mysql container.


## Installation


If you'd like to build the image yourself then:

```bash
$ git clone https://github.com/eugeneware/docker-wordpress-nginx.git
$ cd docker-wordpress-nginx
$ sudo docker build -t="rija/docker-wordpress-nginx" .
```

## Usage

To spawn a new instance of wordpress on port 80.  The -p 80:80 maps the internal docker port 80 to the outside port 80 of the host machine.

```bash
$ docker run --name wordpress  -d -p 80:80 --link mysql:db docker-wordpress-nginx
```

You will need to have started a Mysql container before hand:

```bash
$ docker run --name mysql -e MYSQL_ROOT_PASSWORD=<root password> -e MYSQL_DATABASE=wordpress -e MYSQL_USER=<user name> -e MYSQL_PASSWORD=<user password> -d mysql:5.5.42
```


After starting the docker-wordpress-nginx check to see if it started and the port mapping is correct.  This will also report the port mapping between the docker container and the host machine.

```
$ sudo docker ps

0.0.0.0:80 -> 80/tcp docker-wordpress-nginx
```

You can then visit the following URL in a browser on your host machine to get started:

```
http://127.0.0.1:80
```
