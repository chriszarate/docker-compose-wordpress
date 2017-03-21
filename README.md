# WordPress plugin or theme development with Docker Compose

[![Build status][build-status]][travis-ci]

This is an example repo for how one might wire up Docker Compose with the
[chriszarate/wordpress][image] image for plugin or theme development. In
addition to WP-CLI, PHPUnit, Composer, Xdebug, and the WordPress unit testing
suite, the `docker-compose.yml` file adds MariaDB and `nginx-proxy` to create a
complete development environment that boots up quickly.


## Set up

1. Clone or fork this repo.

2. Put your plugin or theme code in the root of this folder and adjust the 
   `services/wordpress/volumes` section of `docker-compose.yml` so that it
   syncs to the appropriate directory.

   If you would like your plugin or theme activated when the container starts,
   edit the `WORDPRESS_ACTIVATE_PLUGINS` or `WORDPRESS_ACTIVATE_THEME`
   environment variables.

3. Add `project.dev` (or your chosen hostname) to `/etc/hosts`, e.g.:

```
127.0.0.1 localhost project.dev
```

   If you choose a different hostname, edit `.env` as well.


## Start environment

```sh
docker-compose up -d
```

The first time you run this, it will take a few minutes to pull in the required
images. On subsequent runs, it should take less than 30 seconds before you can
connect to WordPress in your browser. (Most of this time is waiting for MariaDB
to be ready to accept connections.)

The `-d` flag backgrounds the process and log output. To view logs for a
specific container, use `docker-compose logs [container]`, e.g.:

```sh
docker-compose logs wordpress
```

Please refer to the [Docker Compose documentation][docker-compose] for more
information about starting, stopping, and interacting with your environment.

Log in to `/wp-admin/` with `wordpress` / `wordpress`.


## Update environment

To pull in the latest images (including `chriszarate/wordpress`), make sure your
clone/fork of this repo is up to date, then run the following commands. Note
that this will **destroy** your current environment, including the database, and
reset it to its initial state.

```sh
docker-compose down
docker-compose pull
docker-compose up -d
```


## WP-CLI

```sh
docker-compose exec wordpress wp [command]
```


## Running tests (PHPUnit)

The tests in this example repo were generated with WP-CLI:

```sh
docker-compose exec wordpress wp scaffold plugin-tests my-plugin
```

This is not required, however, and you can bring your own test scaffold. Either
way, in docker-compose.yml, set the `PHPUNIT_TEST_DIR` environment variable to the path containing
`phpunit.xml`. Tests are run against a separate MariaDB instance to ensure
isolation.

```sh
docker-compose exec wordpress tests
```


## Xdebug

Xdebug is installed but needs the IP of your local machine to connect to your
local debugging client. Edit `.env` and populate the `DOCKER_LOCAL_IP`
environment variable with your machine's (local network) IP address. The default
`idekey` is `xdebug`.

You can enable profiling by appending instructions to `XDEBUG_CONFIG` in
`docker-compose.yml`, e.g.:

```
XDEBUG_CONFIG: "remote_host=${DOCKER_LOCAL_IP} idekey=xdebug profiler_enable=1 profiler_output_name=%R.%t.out"
```

This will output cachegrind files (named after the request URI and timestamp) to
`/tmp` inside the WordPress container.


## Seed MariaDB database

The `mariadb` image supports initializing the database with content by mounting
a volume to the database container at `/docker-entrypoint-initdb.d`. See the
[MariaDB Docker docs][mariadb-docs] for more information.


## Seed `wp-content`

You can seed `wp-content` with files (e.g., an uploads folder) by mounting a
volume to the `wordpress` container at `/tmp/wordpress/init-wp-content`.
Everything in that folder will be copied to your installation's `wp-content`
folder.


[build-status]: https://travis-ci.org/chriszarate/docker-compose-wordpress.svg?branch=master
[travis-ci]: https://travis-ci.org/chriszarate/docker-compose-wordpress
[image]: https://hub.docker.com/r/chriszarate/wordpress/
[docker-compose]: https://docs.docker.com/compose/
[mariadb-docs]: https://github.com/docker-library/docs/tree/master/mariadb#initializing-a-fresh-instance
