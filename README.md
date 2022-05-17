# Peon.dev Docker base images

Images are built automatically using Github actions and pushed to Github Container Registry (ghcr.io).

Purpose of this repository is to speed up builds of other images. This approach is typically called "base images".

## PHP

Contains:
- Composer 2
- Xdebug
- Docker
- [mlocati/php-extension-installer](https://github.com/mlocati/docker-php-extension-installer)

To unload xdebug:
`/usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini`

## PHP Recipes

Contains:
- Composer 2
- Xdebug
- Many many PHP extensions :-).

If you miss any extension, please just [open an issue](https://github.com/peon-dev/docker/issues/new).
