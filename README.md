# docker-mariadb

A Dockerfile that produces a container that will run [MariaDB][mariadb] ,
a drop-in replacement for MySQL.
this dockerfile is based on `paintedfox/mariadb`

[mariadb]: https://mariadb.org/

## Container Creation / Running

The MariaDB server is configured to store data in `/data` inside the container.
You can map the container's `/data` volume to a volume on the host so the data
becomes independant of the running container.

This example uses `/tmp/mariadb` to store the MariaDB data, but you can modify
this to your needs.

When the container runs, it creates a root with a random password.  You
can set the username and password for the root by setting the container's
environment variables.  This lets you discover the username and password of the
root from within a linked container or from the output of `docker inspect
mariadb`.

``` shell
$ mkdir -p /tmp/mariadb
$ docker run -d -name="mariadb" \
             -p 127.0.0.1:3306:3306 \
             -v /tmp/mariadb:/data \
             -e USER="root" \
             -e PASS="$(pwgen -s -1 16)" \
             sudokeys/mariadb
```

## Connecting to the Database

To connect to the MariaDB server, you will need to make sure you have a client.
You can install the `mysql-client` on your host machine by running the
following (Ubuntu 12.04LTS):

``` shell
$ sudo apt-get install mysql-client
```

As part of the startup for MariaDB, the container will generate a random
password for the root.  To view the login in run `docker logs
<container_name>` like so:

``` shell
$ docker logs mariadb
MARIADB_USER=root
MARIADB_PASS=FzNQiroBkTHLX7y4
MARIADB_DATA_DIR=/data
Starting MariaDB...
140103 20:33:49 mysqld_safe Logging to '/data/mysql.log'.
140103 20:33:49 mysqld_safe Starting mysqld daemon with databases from /data
```

Then you can connect to the MariaDB server from the host with the following
command:

``` shell
$ mysql -u root --password=FzNQiroBkTHLX7y4 --protocol=tcp
```

## Linking with the Database Container

You can link a container to the database container.  You may want to do this to
keep web application processes that need to connect to the database in
a separate container.

To demonstrate this, we can spin up a new container like so:

``` shell
$ docker run -t -i -link mariadb:db ubuntu bash
```

This assumes you're already running the database container with the name
*mariadb*.  The `-link mariadb:db` will give the linked container the alias
*db* inside of the new container.

From the new container you can connect to the database by running the following
commands:

``` shell
$ apt-get install -y mysql-client
$ mysql -u "$DB_ENV_USER" --password="$DB_ENV_PASS" -h "$DB_PORT_3306_TCP_ADDR" -P "$DB_PORT_3306_TCP_PORT"
```

If you ran the *mariadb* container with the flags `-e USER=<user>` and `-e
PASS=<pass>`, then the linked container should have these variables available
in its environment.  Since we aliased the database container with the name
*db*, the environment variables from the database container are copied into the
linked container with the prefix `DB_ENV_`.
