# Drupal Dev Image

The Drupal Dev image installs Composer "dev" dependencies and [Xdebug](https://xdebug.org/). When you create your own image you would add whatever dev only dependencies/code needed.

This image extends your normal (releasable) Drupal image. It is built automatically by docker-compose (see [docker-compose.yml](../docker-compose.yml)).

The reason this image builds atop your normal Drupal image is to ensure that your dev tools never enter your releasable build. By keeping this separation you can be sure to have consistent and dependable builds.
