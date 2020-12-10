# Laravel-Websockets + Docker + Caddy example

This repo is meant to show an example of how a project with [Laravel-Websockets](https://github.com/beyondcode/laravel-websockets) run using Docker and using [Caddy](https://caddyserver.com) as the server. 

## Usage

```bash
# Prep
$ cp .env.example .env
$ cp docker/.env.example docker/.env

# To spin up the Docker stack (build will take time)
$ docker/utils up

# To get a shell to play around with PHP, Yarn, etc.
$ docker/utils shell

# Run tests
$ docker/utils test

# See the 'utils' help
$ docker/utils

# Run 'docker-compose ps' on the stack
$ docker/utils dc ps

# Tear it down
$ docker/utils down
```

## Things of note

Take a look at the individual commits to see the general idea of the setup.

This README assumes you already know how to use Laravel, in general.

### Files to look at

- [`docker/utils`](docker/utils)
    - Helper shell script for running the app, quick access to common maintenance tasks, getting a shell into the `workspace` container.
- [`Caddyfile`](docker/caddy/Caddyfile)
    - Runs the Laravel app with php-fpm
    - Proxies any requests with websocket headers to the `websockets` container
    - Serves static files
    - Please ask on https://caddy.community for any Caddy usage questions!
- [`docker-compose.yml`](docker/docker-compose.yml)
    - Service definitions for everything you need to run the app
    - Uses variables from `docker/.env` which can be overridden
    - The `workspace` container gives you somewhere to run your development/maintenance tasks
- [`docker/.env`](docker/.env.example)
    - Change the `CADDY_HOST_LABEL` variable to your domain name to serve the app automatically over HTTPS  with certificates fetched from Let's Encrypt (please read the Caddy docs!)
    - Change the `COMPOSE_PROJECT_NAME` to some better name for your project, is used by `docker-compose` to namespace the container and network names
    - Change the `DB_*` variables to secure your database (generate random passwords!!), and also update your [`.env`](.env.example) with the same values.
        - `DB_DATABASE`
        - `DB_USER`
        - `DB_PASSWORD`
        - `DB_ROOT_PASSWORD`
- [`bootstrap.js`](resources/js/bootstrap.js)
    - Laravel Echo is properly configured to use `laravel-websockets` on the same host/port as the initial request. Will use `wss://` automatically if the site was loaded over `https://`.
