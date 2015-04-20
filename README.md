# docker-wordpress-nginx


This is a fork of [eugeneware's Dockerfile for Wordpress](https://github.com/eugeneware/docker-wordpress-nginx).

My plan is to allow the use of linked and data volume containers and to make it easy for upgrades and backups.

Currently you can link to another Mysql container and use data volume containers.


## Installation

Assuming you already have a mysql container named *mysql-server* running, you can pull the image directly from docker hub and run it with the following command:

```bash
$ docker run --name wordpress -d -p 80:80 --link mysql-server:db rija/wordpress-nginx-no-mysql

```

If you'd like to build the image yourself then:

```bash
$ git clone https://github.com/rija/docker-wordpress-nginx.git
$ cd docker-wordpress-nginx
$ docker build -t="wordpress-nginx-no-mysql" .
```

## Usage

To spawn a new instance of wordpress on port 80.  The -p 80:80 maps the internal docker port 80 to the outside port 80 of the host machine.

The example also use a data volume container to keep the *wp-content* user data folder insulated from future upgrades of the wordpress container.

```bash
$ docker create --name wp-content -v /usr/share/nginx/www/wp-content rija/wordpress-nginx-no-mysql
$ docker run --name wordpress-server --volumes-from wp-content -d -p 80:80 --link mysql-server:db rija/wordpress-nginx-no-mysql
```

You will need to have started a Mysql container beforehand:

```bash
$ docker create --name mysql-data -v /var/lib/mysql mysql:5.5.42
$ docker run --name mysql-server --volumes-from mysql-data -e MYSQL_ROOT_PASSWORD=<root password> -e MYSQL_DATABASE=wordpress -e MYSQL_USER=<user name> -e MYSQL_PASSWORD=<user password> -d mysql:5.5.42
```


After starting the containers check to see if it started and the port mapping is correct.  This will also report the port mapping between the docker container and the host machine.

```
$ docker ps

CONTAINER ID        IMAGE                           COMMAND                CREATED             STATUS              PORTS                NAMES
e1afad973636        wordpress-nginx-no-mysql:latest   "/bin/bash /start.sh   25 minutes ago      Up 25 minutes       0.0.0.0:80->80/tcp   wordpress-server    
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

To access the file system for the wordpress content, you will use the following command;

```bash
$ docker run --rm  --volumes-from wp-content -it rija/wordpress-nginx-no-mysql  /bin/bash
```

To make a manual connection to the mysql database, you make a shell connection to ephemeral container that's linked to the mysql database container:

```bash
docker run --rm --link mysql-server:wdb -it rija/wordpress-nginx-no-mysql  bash
```

From there you can get access to the database connection strings and connect to the mysql using the command line client:

```bash
$ env

HOSTNAME=8112dabbc654
WDB_ENV_MYSQL_DATABASE=wordpress
TERM=xterm
...
WDB_PORT=tcp://172.17.0.2:3306
WDB_ENV_MYSQL_USER=wordpress
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
WDB_ENV_MYSQL_ROOT_PASSWORD=therootpassword
PWD=/
WDB_PORT_3306_TCP_ADDR=172.17.0.2
WDB_PORT_3306_TCP=tcp://172.17.0.2:3306
WDB_NAME=/kickass_heisenberg/wdb
WDB_PORT_3306_TCP_PROTO=tcp
WDB_ENV_MYSQL_MAJOR=5.5
SHLVL=1
HOME=/root
WDB_ENV_MYSQL_VERSION=5.5.42
DEBIAN_FRONTEND=noninteractive
LESSOPEN=| /usr/bin/lesspipe %s
WDB_ENV_MYSQL_PASSWORD=wordpress
LESSCLOSE=/usr/bin/lesspipe %s %s
WDB_PORT_3306_TCP_PORT=3306

$ mysql -h $WDB_PORT_3306_TCP_ADDR -u $WDB_ENV_MYSQL_USER -p $WDB_ENV_MYSQL_DATABASE

```


## Migrating Data Volume Containers across hosts

The scenario is as follow:
You have deployed Wordpress on a laptop following the instructions above
You have created content (published blog posts, installed theme and plugins)

Now you want to deploy the whole thing on a test server for your client to see.


How do you do that?

Below is how you do it for wp-content Data Volume Container. It works the same for mysql-data Volume Container. 

On your laptop:

```bash
$ docker ps -a

cdb4f596dcc5        rija/wordpress-nginx-no-mysql:latest   "/bin/bash /start.sh   2 days ago          Up 2 days           0.0.0.0:80->80/tcp   wordpress-server    
076f0ab9098e        mysql:5.5.42                           "/entrypoint.sh mysq   2 weeks ago         Up 2 days           3306/tcp             mysql-server        
root@hkims-test:~# docker ps -a
CONTAINER ID        IMAGE                                  COMMAND                CREATED             STATUS                     PORTS                NAMES
cdb4f596dcc5        rija/wordpress-nginx-no-mysql:latest   "/bin/bash /start.sh   2 days ago          Up 2 days                  0.0.0.0:80->80/tcp   wordpress-server         
d5fe2b4f8cee        rija/wordpress-nginx-no-mysql:latest   "/bin/bash /start.sh   2 weeks ago                                                         wp-content               
076f0ab9098e        mysql:5.5.42                           "/entrypoint.sh mysq   2 weeks ago         Up 2 days                  3306/tcp             mysql-server             
6dec6ae9b207        mysql:5.5.42                           "/entrypoint.sh mysq   2 weeks ago                                                         mysql-data               



$ docker commit -m "wp-content data" d5fe2b4f8cee wp-content-data


4f177bd27a9ff0f6dc2a830403925b5360bfe0b93d476f7fc3231110e7f71b1c

$ docker images

REPOSITORY                      TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
wp-content-data                 latest              68e2ee6c238f        4 seconds ago       521.1 MB
rija/wordpress-nginx-no-mysql   latest              efa65bbde74c        2 weeks ago         521.1 MB
<none>                          <none>              f02e3c203c86        2 weeks ago         521 MB
<none>                          <none>              2b486da790be        2 weeks ago         521.1 MB
docker-wordpress-nginx          latest              1dbcd3deaab9        2 weeks ago         521 MB


$ docker save -o /tmp/wp-content-data.tar 68e2ee6c238f

```

Then you copy the tar image to the server you want to deploy 

and run the following command

```bash

docker load -i ~/wp-content-data.tar

docker create --name wp-content -v /usr/share/nginx/www/wp-content wp-content-data


```

Once you've done this for the mysql database as well, you can run (with docker run) the wordpress and mysql containers as described in the Usage section
