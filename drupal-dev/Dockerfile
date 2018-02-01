# This Dockerfile adds dev requirements to the Docker image built in ../drupal.
#
# You will need to build that image before being able to build this image.
# `docker-compose up` will do this on local, your build pipeline should do this
# in your CD environment.

# The tag to use as the base image. In your local this would be `dev`, in the CD
# build this would be the commit SHA (per the ../.drone.yml config).
ARG UPSTREAM_TAG

FROM aaadigital/drupal-starter:${UPSTREAM_TAG} AS base

# Use Composer to install dev-dependencies.
FROM composer:1.5 AS composer
# Bring all of the build files from the upstream image into Composer.
COPY --from=base /var/www/html /app
WORKDIR /app
# Install additional dev dependencies with Composer.
RUN composer install

FROM aaadigital/drupal-starter:${UPSTREAM_TAG}
COPY --from=composer /app /var/www/html
# Since we're in a multi-stage build our chown'd file permissions don't persist
# from the upstream drupal-starter image, thus we re-apply.
RUN chown -R www-data:www-data sites modules themes ../config
# Install PHP extensions we want for dev only. Extensions for your releasable
# image are defined in ../drupal/Dockerfile.
RUN pecl install xdebug-2.5.5 \
	&& docker-php-ext-enable xdebug
