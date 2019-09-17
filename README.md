Code Colossus
=============

Simple Middleman blog for Code Colossus (Software Development/Tech).

## Setup

* Configure a config/deploy.yml from config/deploy.yml.sample

## Deploying

Hosting is handled by dokku as a static site with the [buildpack-nginx](https://github.com/dokku/buildpack-nginx) buildpack (specified in `.env`).

Configuration:

    NGINX_ROOT=build
