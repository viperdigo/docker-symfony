# Docker stack for Symfony projects

:octocat:

[![Build Status](https://travis-ci.org/viperdigo/docker-symfony.svg?branch=symfony4&style=flat-square)](https://travis-ci.org/viperdigo/docker-symfony)
[![license](https://img.shields.io/github/license/mashape/apistatus.svg?style=flat-square)](LICENSE)
[![contributions](https://img.shields.io/badge/contributions-welcome-brightgreen.svg?style=flat-square)](https://github.com/viperdigo/docker-symfony/issues)
[![HitCount](http://hits.dwyl.io/viperdigo/docker-symfony.svg)](http://hits.dwyl.io/viperdigo/docker-symfony)

![](doc/schema.png)

## Project Based in 
([docker-for-symfony](https://github.com/carlosas/docker-for-symfony/tree/symfony-4)):

The idea of this project is to follow this project but for symfony2, symfony3 and symfony4 and laravel 5+

## Basic info

* [nginx](https://nginx.org/)
* [PHP-FPM](https://php-fpm.org/)
* [MySQL](https://www.mysql.com/)
* [Redis](https://redis.io/)
* [Elasticsearch](https://www.elastic.co/products/elasticsearch)
* [Logstash](https://www.elastic.co/products/logstash)
* [Kibana](https://www.elastic.co/products/kibana)
* [RabbitMQ](https://www.rabbitmq.com/)

## Previous requirements

This stack needs [docker](https://www.docker.com/) and [docker-compose](https://docs.docker.com/compose/) to be installed.

## Installation

1. Create a `.env` file from `.env.dist` and adapt it according to the needs of the application

    ```sh
    $ cp .env.dist .env && nano .env
    ```

2.  Due to an Elasticsearch 6 requirement, we may need to set a host's sysctl option and restart ([More info](https://github.com/spujadas/elk-docker/issues/92)):

    ```sh
    $ sudo sysctl -w vm.max_map_count=262144
    ```

3. Build and run the stack in detached mode (stop any system's ngixn/apache2 service first)

    ```sh
    $ docker-compose build
    $ docker-compose up -d
    ```

4. Get the bridge IP address

    ```sh
    $ docker network inspect bridge | grep Gateway | grep -o -E '[0-9\.]+'
    # OR an alternative command
    $ ifconfig docker0 | awk '/inet:/{ print substr($2,6); exit }'
    # OR try 127.0.0.1
    ```

5. Update your system's hosts file with the IP retrieved in **step 4**

    ```sh
    # OSX/Linux
    $ sudo vi /etc/hosts
    # Windows
    edit with adm permission the file 'c:\Windows\System32\Drivers\etc\hosts'

6. Prepare the Symfony application
    1. Update Symfony env variables (*.env*)

        ```
        #...
        DATABASE_URL=mysql://db_user:db_password@mysql:3306/db_name
        #...
        ```

    2. Composer install & update the schema from the container

        ```sh
        $ docker-compose exec php bash
        $ composer install
        $ symfony doctrine:schema:update --force
        ```
7. (Optional) Xdebug: Configure your IDE to connect to port `9001` with key `PHPSTORM`

## How does it work?

We have the following *docker-compose* built images:

* `nginx`: The Nginx webserver container in which the application volume is mounted.
* `php`: The PHP-FPM container in which the application volume is mounted too.
* `mysql`: The MySQL database container.
* `elk`: Container which uses Logstash to collect logs, send them into Elasticsearch and visualize them with Kibana.
* `redis`: The Redis server container.
* `rabbitmq`: The RabbitMQ server/administration container.

Running `docker-compose ps` should result in the following running containers:

```
      Name                    Command               State                                             Ports
------------------------------------------------------------------------------------------------------------------------------------------------------
container_elk      /usr/local/bin/start.sh          Up      0.0.0.0:5044->5044/tcp, 0.0.0.0:5601->5601/tcp, 0.0.0.0:9200->9200/tcp, 9300/tcp
container_mysql    docker-entrypoint.sh mysqld      Up      0.0.0.0:3306->3306/tcp, 33060/tcp
container_nginx    nginx                            Up      443/tcp, 0.0.0.0:80->80/tcp
container_php      docker-php-entrypoint php-fpm    Up      9000/tcp
container_rabbit   docker-entrypoint.sh rabbi ...   Up      15671/tcp, 0.0.0.0:15672->15672/tcp, 25672/tcp, 4369/tcp, 5671/tcp, 0.0.0.0:5672->5672/tcp
container_redis    docker-entrypoint.sh redis ...   Up      0.0.0.0:6379->6379/tcp
```

## Usage

Once all the containers are up, our services are available at:

* Symfony app: `http://symfony.local:80`
* Mysql server: `symfony.local:3306`
* Redis: `symfony.local:6379`
* Elasticsearch: `symfony.local:9200`
* Kibana: `http://symfony.local:5601`
* RabbitMQ: `http://symfony.local:15672`
* Log files location: *logs/nginx* and *logs/symfony*

---

:tada: Now we can stop our stack with `docker-compose down` and start it again with `docker-compose up -d`
