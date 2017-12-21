# This is a multi-stage build, you can read about how they work:
# https://docs.docker.com/engine/userguide/eng-image/multistage-build/

# This is setting the default Drupal version to 8.4 it can be overridden in a
# `docker build` or in your docker-compose.yml in the base of the repo.
ARG DRUPAL_VERSION=8.4

# Define a base image to work from, this isn't needed if you maintain your
# own composer.json/lock.
FROM drupal:${DRUPAL_VERSION}-apache AS base

# Use Composer to install dependencies.
FROM composer:1.5 AS composer
# Bring all of the build files from the Drupal image into Composer.
COPY --from=base /var/www/html /app
# The next line copies files from the build context into Composer, potentially
# overwriting conflicting files.
#
# This is particularly important should the build context have a
# composer.json/lock, in which case your Drupal Core/dependencies will end up
# matching what is contained in the composer.json/lock rather than the Drupal
# Docker image we copied from in the prior step.
#
# You can use .dockerignore to ignore files from the build context's copy.
COPY ./html /app
WORKDIR /app
# Install dependencies with Composer.
RUN composer install --no-dev

FROM drupal:${DRUPAL_VERSION}-apache
COPY --from=composer /app /var/www/html
# Best practice is to store config exports outside the webroot.
#
# Be sure to update your settings.php to include:
# $config_directories[CONFIG_SYNC_DIRECTORY] = '../config/sync';
COPY ./config ../config
