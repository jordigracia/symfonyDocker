# Docker for Symfony v.2/3 Application 
symfonyDocker it's a basic setu up repo that gives you everything you need for developing Symfony application under docker containers. 

## Containers
* PHP7.1-FPM
* NGINX
* MySQL
* ELK
* KIBANA


## Install Docker
First off all you'll need to install docker on your system.
[Docker Install](https://docs.docker.com/install/#supported-platforms)


## Install Docker Containers

1. Create a `.env` from the `.env.dist` file. Adapt it according to your symfony application
change path to your symfony local application path

    ```bash
    cp .env.dist .env
    ```


2. Build/run containers with (with and without detached mode)

    ```bash
    $ docker-compose build
    $ docker-compose up -d (detached mode) or docker-compose up
    ```

3. Update your system host file (add symfony.local) or (nameyoutapplication.local)

    ```bash
    # UNIX only: get containers IP address and update host (replace IP according to your configuration) 
    $ sudo echo $(docker network inspect bridge | grep Gateway | grep -o -E '[0-9\.]+') "symfony.local" >> /etc/hosts
    # Windows, edit C:\Windows\System32\drivers\etc\hosts
    ```
    
    **Note:** For **OS X**, please take a look [here](https://docs.docker.com/docker-for-mac/networking/) and for **Windows** read [this](https://docs.docker.com/docker-for-windows/#/step-4-explore-the-application-and-run-examples) (4th step).

4. Prepare your Symfony app
    1. Update app/config/parameters.yml

        ```yml
        # path/to/your/symfony-project/app/config/parameters.yml
        parameters:
            database_host: db
        ```

    2. Composer install

        ```bash
        $ docker-compose exec php bash
        $ composer install
        ```
        
    3. Create Symfony DB
        
        #### For Symfony2 Applications
        ```bash
        $ sf doctrine:database:create
        $ sf doctrine:schema:update --force
        # Only if you have `doctrine/doctrine-fixtures-bundle` installed
        $ sf doctrine:fixtures:load --no-interaction
        ```
        
        #### For Symfony3 Applications
        ```bash
        $ sf3 doctrine:database:create
        $ sf3 doctrine:schema:update --force
        # Only if you have `doctrine/doctrine-fixtures-bundle` installed
        $ sf3 doctrine:fixtures:load --no-interaction
        ```

## Usage

Just run `docker-compose up -d`, then:

* Symfony app: visit [symfony.local](http://symfony.local)  
* Symfony dev mode: visit [symfony.local/app_dev.php](http://symfony.local/app_dev.php)  
* Logs (Kibana): [symfony.local:81](http://symfony.local:81)
* Logs (files location): logs/nginx and logs/symfony
* phpMyAdmin: visit [symfony.local:8080](http://symfony.local:8080)


## How it works?

Have a look at the `docker-compose.yml` file, here are the `docker-compose` built images:

* `db`: This is the MySQL database container, remember to reconfigure parameters.yml on your Symfony app.
* `php`: This is the PHP-FPM 7.1 container in which the application volume is mounted, in this container we can execute composer commands
* `nginx`: This is the Nginx webserver container
* `elk`: This is a ELK stack container which uses Logstash to collect logs, send them into Elasticsearch and visualize them with Kibana.


This results in the following running containers:

```bash
$ docker-compose ps
```

```bash
Name            Command                          State   Ports            
--------------------------------------------------------------------------------------------------
db_1            /entrypoint.sh mysqld            Up      0.0.0.0:3306->3306/tcp      
elk_1           /usr/bin/supervisord -n -c ...   Up      0.0.0.0:81->80/tcp          
nginx_1         nginx                            Up      443/tcp, 0.0.0.0:80->80/tcp
php_1           php-fpm                          Up      0.0.0.0:9000->9000/tcp      
```

## Useful Docker Commands

```bash
# To Execute Bash commands:
# Php is the name of container defined on docker-compose.yml
$ docker-compose exec php bash
# If we want to connect to Nginx contanier bash
$ docker-compose exec nginx bash
```

```bash
# Composer commands
$ docker-compose exec php composer install
$ docker-compose exec php composer update
```

```bash
# Simfony commands (Tips: there is an alias inside php container sf)
# Clear Cache
$ docker-compose exec php php /var/www/symfony/app/console cache:clear # Symfony2
$ docker-compose exec php php /var/www/symfony/bin/console cache:clear # Symfony3
```

```bash
# Same command by using alias
$ docker-compose exec php bash
$ sf cache:clear
```

```bash
# MySQL commands/querys, first of all connect to mysql Container
$ docker-compose exec db mysql -uroot -p"root"
```

```bash
# Check CPU consumption
$ docker stats
```

```bash
# Delete all containers
$ docker rm $(docker ps -aq)
```

```bash
# Delete all images
$ docker rmi $(docker images -q)
```
