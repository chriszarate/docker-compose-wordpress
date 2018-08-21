# WordPress plugin or theme development with Docker Compose

[![Build status][build-status]][travis-ci]

This is an example repo for how one might wire up Docker Compose for local
plugin or theme development. It provides WordPress, MariaDB, WP-CLI, PHPUnit,
and the WordPress unit testing suite.


## Set up

1. Clone or fork this repo.

2. Put your plugin or theme code in the root of this folder and adjust the 
   `services/wordpress/volumes` section of `docker-compose.yml` so that it
   syncs to the appropriate directory.

3. Add `project.test` to `/etc/hosts`, e.g.:

   ```
   127.0.0.1 localhost project.test
   ```


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


## Install WordPress

```sh
docker-compose run --rm wp-cli install-wp
```

Log in to `http://project.test/wp-admin/` with `wordpress` / `wordpress`.

Alternatively, you can navigate to `http://project.test/` and manually perform
the famous five-second install.


## WP-CLI

You will probably want to [create a shell alias][3] for this:

```sh
docker-compose run --rm wp-cli wp [command]
```


## Running tests (PHPUnit)

The tests in this example repo were generated with WP-CLI, e.g.:

```sh
docker-compose run --rm wp-cli wp scaffold plugin-tests my-plugin
```

This is not required, however, and you can bring your own test scaffold. The
important thing is that you provide a script to install your test dependencies,
and that these dependencies are staged in `/tmp`.

The testing environment is provided by a separate Docker Compose file
(`docker-compose.phpunit.yml`) to ensure isolation. To use it, you must first
start it, then manually run your test installation script. These commands work
for this example repo, but may not work for you if you use a different test
scaffold.

Note that, in the PHPUnit container, your code is mapped to `/app`.

```sh
docker-compose -f docker-compose.yml -f docker-compose.phpunit.yml up -d
docker-compose -f docker-compose.phpunit.yml run --rm wordpress_phpunit /app/bin/install-wp-tests.sh wordpress_test root '' mysql_phpunit latest true
```

Now you are ready to run PHPUnit. Repeat this command as necessary:

```sh
docker-compose -f docker-compose.phpunit.yml run --rm wordpress_phpunit phpunit
```


## Changing the hostname

You can change the hostname from the default `project.test` by adding a `.env`
file at the project root and defining the `DOCKER_DEV_DOMAIN` environment
variable:

```
DOCKER_DEV_DOMAIN=myproject.test
```


## Seed MariaDB database

The `mariadb` image supports initializing the database with content by mounting
a volume to the database container at `/docker-entrypoint-initdb.d`. See the
[MariaDB Docker docs][mariadb-docs] for more information.


## Troubleshooting

If your stack is not responding, the most likely cause is that a container has
stopped or failed to start. Check to see if all of the containers are "Up":

```
docker-compose ps
```

If not, inspect the logs for that container, e.g.:

```
docker-compose logs wordpress
```


[build-status]: https://travis-ci.org/chriszarate/docker-compose-wordpress.svg?branch=master
[travis-ci]: https://travis-ci.org/chriszarate/docker-compose-wordpress
[docker-compose]: https://docs.docker.com/compose/
[mariadb-docs]: https://github.com/docker-library/docs/tree/master/mariadb#initializing-a-fresh-instance
