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

## (Re)Build Docker Images

```bash
docker-compose build
```

## Run Containers Locally

Run in the foreground:

```bash
docker-compose up
```

Run in the background:

```bash
docker-compose up -d
```

Stop/remove containers:

```bash
docker-compose down
```

See the [docker-compose CLI overview](https://docs.docker.com/compose/reference/overview/) for other commands.

### Host Accessible Endpoints

- Drupal: http://localhost:8080/

    _Web head for Drupal, in a production environment you'd likely put a reverse proxy in front of these containers._

You can change any of the above ports, without changing any git tracked files, by using environment variables. For example, to have the Drupal container accessible on port 8112 you'd run: `DRUPAL_PORT=8112 docker-compose up`.

If desired, you can even create a .env file in the repo root with this format to avoid specifying ports on every command:

```ini
DRUPAL_PORT=8080
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
