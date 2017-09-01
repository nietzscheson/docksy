It's inspired completly in [docker-symfony](https://github.com/maxpou/docker-symfony).

# Docker Symfony (PHP7-FPM - NGINX - MySQL - Docker-Sync)

## Installation

1. Install and setup [docker-sync](https://github.com/EugenMayer/docker-sync). See the [Wiki](https://github.com/EugenMayer/docker-sync/wiki)

2. Create a `.env` from the `.env.dist` file. Adapt it according to your symfony application

    ```bash
    cp .env.dist .env
    ```
3. Build/run containers with (with and without detached mode)

    ```bash
    $ docker-sync-stack start
    ```
4. Update your system host file (add docksy.dev) // change VIRTUAL_HOST value in .env

    ```bash
    # UNIX only: get containers IP address and update host (replace IP according to your configuration) (on Windows, edit C:\Windows\System32\drivers\etc\hosts)
    $ sudo echo $(docker network inspect bridge | grep Gateway | grep -o -E '[0-9\.]+') "docksy.dev" >> /etc/hosts
    ```

    **Note:** For **OS X**, please take a look [here](https://docs.docker.com/docker-for-mac/networking/) and for **Windows** read [this](https://docs.docker.com/docker-for-windows/#/step-4-explore-the-application-and-run-examples) (4th step).

5. Prepare Symfony app
    1. Update app/config/parameters.yml

        ```yml
        # path/to/your/symfony-project/app/config/parameters.yml
        parameters:
            database_host: mysql
        ```

    2. Composer install & create database

        ```bash
        $ docker-compose exec php bash
        $ composer install
        # Symfony2 | Symfony 3
        $ [php app/console | bin/console] doctrine:database:create
        $ [php app/console | bin/console] doctrine:schema:update --force
        # Only if you have `doctrine/doctrine-fixtures-bundle` installed
        $ [php app/console | bin/console] doctrine:fixtures:load --no-interaction
        ```

6. Enjoy :-)

## Usage

Just run `docker-compose up -d`, then:

* Symfony app: visit [docksy.dev](http://symfony.dev)  
* Symfony dev mode: visit [docksy.dev/app_dev.php](http://symfony.dev/app_dev.php)  

## Customize

If you want to add optionnals containers like Redis, PHPMyAdmin... take a look on [doc/custom.md](https://github.com/maxpou/docker-symfony/blob/master/doc/custom.md).

## How it works?

Have a look at the `docker-compose.yml` file, here are the `docker-compose` built images:

* `mysql`: This is the MySQL database container,
* `php`: This is the PHP-FPM container in which the application volume is mounted,
* `nginx`: This is the Nginx webserver container in which application volume is mounted too,

This results in the following running containers:

```bash
$ docker-compose ps
           Name                          Command               State              Ports            
--------------------------------------------------------------------------------------------------
docksy_db_1                       entrypoint.sh mysqld          Up      0.0.0.0:3306->3306/tcp      
docksy_nginx_1                    nginx                         Up      443/tcp, 0.0.0.0:80->80/tcp
docksy_php_1                      php-fpm                       Up      0.0.0.0:9000->9000/tcp      
docksy-sync                       eugenmayer/unison             Up      500/tcp, 192.168.99.100:32768->5000/tcp 


```

## Useful commands

```bash
# bash commands
$ docker-compose exec php bash

# Composer (e.g. composer update)
$ docker-compose exec php composer update

# SF commands (Tips: there is an alias inside php container)
$ docker-compose exec php php /var/www/symfony/app/console cache:clear # Symfony2
$ docker-compose exec php php /var/www/symfony/bin/console cache:clear # Symfony3
# Same command by using alias
$ docker-compose exec php bash
$ php app/console | bin/console cache:clear | -e=prod

# Retrieve an IP Address (here for the nginx container)
$ docker inspect --format '{{ .NetworkSettings.Networks.dockersymfony_default.IPAddress }}' $(docker ps -f name=nginx -q)
$ docker inspect $(docker ps -f name=nginx -q) | grep IPAddress

# MySQL commands
$ docker-compose exec db mysql -uroot -p"root"

# F***ing cache/logs folder
$ sudo chmod -R 777 app/cache app/logs # Symfony2
$ sudo chmod -R 777 var/cache var/logs var/sessions # Symfony3

# Check CPU consumption
$ docker stats $(docker inspect -f "{{ .Name }}" $(docker ps -q))

# Delete all containers
$ docker rm $(docker ps -aq)

# Delete all images
$ docker rmi $(docker images -q)
```

## FAQ

* Got this error: `ERROR: Couldn't connect to Docker daemon at http+docker://localunixsocket - is it running?
If it's at a non-standard location, specify the URL with the DOCKER_HOST environment variable.` ?  
Run `docker-compose up -d` instead.

* Permission problem? See:
 - [this doc (Setting up Permission)](http://symfony.com/doc/current/book/installation.html#checking-symfony-application-configuration-and-setup)

* [Overwrite the cache and log files outside the shared environment](https://symfony.com/doc/current/configuration/override_dir_structure.html#override-cache-dir). See this [post](https://beberlei.de/2013/08/19/speedup_symfony2_on_vagrant_boxes.html)

* How to config Xdebug? Xdebug is configured out of the box! Just config your IDE to connect port  `9001` and id key `PHPSTORM`

## Contributing

First of all, **thank you** for contributing ♥  
If you find any typo/misconfiguration/... please send me a PR or open an issue. You can also ping me on [twitter](https://twitter.com/nietzscheson).  
Also, while creating your Pull Request on GitHub, please write a description which gives the context and/or explains why you are creating it.

## Bibliography
* [Dockerise your PHP application with Nginx and PHP7-FPM](http://geekyplatypus.com/dockerise-your-php-application-with-nginx-and-php7-fpm/)
* [Making your dockerised PHP application even better](http://geekyplatypus.com/making-your-dockerised-php-application-even-better/)
* [3 ways to get Docker for Mac faster on your Symfony app](http://blog.michaelperrin.fr/2017/04/14/docker-for-mac-on-a-symfony-app/)
* [docker-sync](http://docker-sync.io/)
* [Symfony Skeleton with docker-sync](https://github.com/timglabisch/symfony_docker_sync)
