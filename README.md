# Docker Drupal Starter

A Docker starter to provide a working example of how one may organize a Drupal project, so that it is compatible with Continuous Delivery.

A secondary goal of the project is to act as a reference/training tool for developers getting up to speed with Docker and CD.

![Build Status](https://webtools.calstate.aaa.com/api/badges/aaa-ncnu/docker-drupal-starter/status.svg)

## Requisites

1. Clone the repo
1. Install Docker
    - macOS:
        1. [Install Docker Engine](https://docs.docker.com/docker-for-mac/)
    - Linux:
        1. [Install Docker Engine](https://docs.docker.com/engine/getstarted/)
        1. [Install Docker Compose](https://docs.docker.com/compose/install/)
        1. The rest of this guide assumes you have performed the actions to [mange Docker as a non-root user](https://docs.docker.com/engine/installation/linux/linux-postinstall/#manage-docker-as-a-non-root-user). If you haven't, you may need to run commands with `sudo` or as a root user.
    - Windows (10+ Pro):
        1. [Enable Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v)
        1. [Install Docker Engine](https://docs.docker.com/docker-for-windows/)
        1. Note, I do not have a Windows 10 Pro machine to test all of this, however I'm applying what I believe are the correct steps. If you find that there are issues or that everything works, please let me know.
1. Create a named network in Docker:

    ```bash
    docker network create traefik
    ```

    _Note you may change this name to anything you want. To do so, you'll need to update the references in docker-compose.yml_

    We create a network so that we can run a container called [Træfik](https://traefik.io/) on it. This container is a reverse proxy that allows us to expose multiple containers to the host machine via a single port by using name routing.

    For example, if you want to run two different web containers on port 80, you can expose only one port on your host machine and use Træfik to access them via different domain names.

1. Generate SSL certificates for your local environment:
    - Mac/Linux:

        ```bash
        docker run --rm \
        -v ${PWD}/traefik:/traefik \
        -w /traefik \
        svagi/openssl req \
        -subj '/CN=*.localhost/O=Local Traefik/C=US' \
        -newkey rsa:2048 -nodes -keyout traefik.key \
        -x509 -days 3650 -out traefik.crt
        ```

    - Windows:

        ```bash
        docker run --rm `
        -v ${PWD}\traefik:/traefik `
        -w /traefik `
        svagi/openssl req `
        -subj '/CN=*.localhost/O=Local Traefik/C=US' `
        -newkey rsa:2048 -nodes -keyout traefik.key `
        -x509 -days 3650 -out traefik.crt
        ```

1. Ensure your machine has the following ports open (you can find this out by continuing in the steps and see if you encounter errors when running the Træfik container):
    - 443: To handle https requests.

        _If you really don't want to use https in your local, you could instead use port 80, but you'll need to make configuration changes in docker-compose.yml to assign the label `"traefik.frontend.entryPoints=http"` for any containers you wish to access via port 80._

    - 8080: To access the Træfik Web UI.
    - Note, you can modify any of these ports as you see fit.

        However, you'll need to specify the port mapping in your traefik/traefik.toml configuration, modify your Traefik run command, and make any connections from your local over the configured port.

1. Run a [Træfik](https://traefik.io/) container with the desired host ports listening:

    - Mac/Linux:

        ```bash
        docker run --rm -d \
        --network=traefik \
        -p "443:443" \
        -p "8080:8080" \
        -v /var/run/docker.sock:/var/run/docker.sock \
        -v ${PWD}/traefik/traefik.toml:/etc/traefik/traefik.toml \
        -v ${PWD}/traefik/traefik.crt:/etc/ssl/traefik.crt \
        -v ${PWD}/traefik/traefik.key:/etc/ssl/traefik.key \
        traefik
        ```

    - Windows:

        ```bash
        docker run --rm -d `
        --network=traefik `
        -p "443:443" `
        -p "8080:8080" `
        -v /var/run/docker.sock:/var/run/docker.sock `
        -v ${PWD}\traefik\traefik.toml:/etc/traefik/traefik.toml `
        -v ${PWD}\traefik\traefik.crt:/etc/ssl/traefik.crt `
        -v ${PWD}\traefik\traefik.key:/etc/ssl/traefik.key `
        traefik
        ```

1. Since we use name resolution in our reverse proxy (Træfik), domains under *.localhost need to be configured to resolve to your local machine (127.0.0.1):

    This can be accomplished by modifying the hosts file on your system to hardcode each domain to 127.0.0.1 or by setting up a local DNS to wildcard *.localhost - such as [dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html) (Mac/Linux) or [Acrylic](http://mayakron.altervista.org/wikibase/show.php?id=AcrylicHome) (Windows)

    Domains to resolve at 127.0.0.1:

    - drupal.localhost
    - dev.drupal.localhost
    - adminer.drupal.localhost

    _Note that some web browsers (Chromium based and possible others) will resolve *.localhost to 127.0.0.1 automatically._

## (Re)Build Docker Images

```bash
docker-compose build
```

## Run Containers Locally

Run in the foreground:

```bash
docker-compose up --build
```

_The --build flag forces previously built Docker images to rebuild._

Run in the background:

```bash
docker-compose up --build -d
```

Stop/remove containers/volumes:

```bash
docker-compose down -v
```

See the [docker-compose CLI overview](https://docs.docker.com/compose/reference/overview/) for other commands.

### Default Host Accessible Endpoints

- Drupal: https://drupal.localhost/

    Container running the image built from ./drupal. The container is not automatically updated as changes are made to the ./drupal/html and ./drupal/config directories. To see changes, an image rebuild is required.

    Rebuild image without taking the whole docker-compose stack down:

    ```bash
    docker-compose rm -fsv drupal
    docker-compose up --build -d drupal
    ```

    - Port 443  
    Port for https.

- Drupal Dev: https://dev.drupal.localhost/

    Like "Drupal" but automatically receives updates from file modifications in ./drupal/html and ./drupal/config.

    Image built from ./drupal-dev.

    - Port 443  
    Port for https.

- Adminer: https://adminer.drupal.localhost/

    [Adminer](https://www.adminer.org/) web interface for MySQL.

    _Note, there's nothing that prevents you gaining CLI access to the MySQL container: `docker-compose exec db mysql ...`._

    - Port 443  
    Port for https.

- Træfik Web UI: http://localhost:8080/

    User interface for the Træfik reverse proxy.

### Installing Drupal

To install Drupal:

1. Visit https://dev.drupal.localhost or https://drupal.localhost (depending if you want to install to your dev site or not)
1. Proceed to the "Set up database" step
1. Fill out the form:
    - Database type: MySQL
    - Database name: _Use whatever you'd like_
    - Database username: root
    - Database password: youshouldmodifythis _(or if you set an environment variable MYSQL_ROOT_PASSWORD then whatever value that is set to)_
    - Advanced options:
        - Host: db
        - Port number: 3306
        - Table name prefix: _Blank, unless you desire one_
1. Save and proceed with the rest of your configuration

### Useful Commands and Examples

- Run Composer commands:

    ```bash
    docker-compose run --rm --no-deps composer [COMPOSER COMMAND]
    ```

    _Note the composer service does not continually run in the background (to reduce resource consumption), so we use the `docker-compose run` command instead of `docker-compose exec`._

    - Composer install a Drupal module ([Paragraphs](https://www.drupal.org/project/paragraphs) in this case):

        ```bash
        docker-compose run --rm --no-deps composer require drupal/paragraphs
        ```

    - Composer install Drush:

        ```bash
        docker-compose run --rm --no-deps composer require drush/drush
        ```


- Run Drush commands (install Drush with Composer first):

    ```bash
    docker-compose exec drupal_dev vendor/bin/drush [DRUSH COMMAND]
    ```

    _Note, many Drush commands require Drupal to be installed. If you encounter errors ensure you have a functional Drupal site if the command requires one._

    - Example, [drupal-directory](https://drushcommands.com/drush-8x/core/drupal-directory/) (dd):

        ```bash
        docker-compose exec drupal_dev vendor/bin/drush dd
        ```

## Project Organization

Directories within the repo root represent Docker images. While each Docker image could potentially be its own Git repository, this project has decided to use directories to clearly represent the relationships of the containers.

 Your local environment needs may not match your production environment. You may decide to run a mail server/client such as [MailHog](https://github.com/mailhog/MailHog) in lower environments, while in production you may instead use a hosted service. Another common example is running your DB in a container in lower environments, while using a managed DB in production.

### Local Environment

At the top level of the project sits a docker-compose.yml file. This file is used to run the project locally and can (should?) be used to influence your container structure in non-dev environments.  
[Learn about Docker Compose](https://docs.docker.com/compose/)

### Continuous Delivery

A .drone.yml file has been included to provide a sample of a Continuous Delivery build configuration. While this file is only usable if you use [Drone](https://github.com/drone/drone), it can act as a guide for your CD tool's configuration.

### Deployments

No-assumptions have been made regarding production deployment strategies since there are many options available. Essentially, the Docker images built in the Continuous Delivery pipeline would be deployed to the production hosting infrastructure, whether that is [Kubernetes](https://kubernetes.io/), [Mesos + Marathon](https://mesosphere.github.io/marathon/), or some other orchestration/hosting method/service.

Your deployment steps would ideally be part of your CD tool's configuration, in this case, the .drone.yml file.

It is likely in production that you may use hosted services to replace some of your containers in lower environments. For example, you may use a managed DB service provided by your host, such as: [GCP's Cloud SQL](https://cloud.google.com/sql/), [Azure's SQL Database](https://azure.microsoft.com/en-us/services/sql-database/), or [AWS's RDS](https://aws.amazon.com/rds/).
