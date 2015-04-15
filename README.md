# Mysql from backup plugin

This plugin is for development deployments when a sample database is needed. The general idea is:

 * You have a mysqldump of a sample/generic database (this is important, since plugin expects this exact format to extract target database name)
 * you only use short project names instead of FQDN-s in dokku. this is important, since the backup file name is based on app name
 * You may push multiple version of your app easily (for Q&A, testing, etc)

Given project „take-over-the-world”, the plugin will:

 * Look for `take-over-the-world.sql` in `$BACKUP_DIR` (and exit if not found)
 * Extract database name from this dump
 * Create a `data.mysql.take-over-the-world` container for volume persistence
 * Launch `mysql.take-over-the-world` container from `$DOCKER_MYSQL_IMAGE` (official `mysql` image by default) using your `$MYSQL_ROOT_PASSWORD`
 * Create the database and import the mysqldump into it, anonymizing emails on the way (`.invalid` is added to all domains except `$EMAIL_DOMAIN_WHITELIST`) — this will be done only once per app. 
 * Set up application configuration to point to this mysql instance (`MYSQL_HOST`, `MYSQL_USERNAME`, `MYSQL_PASSWORD`, `MYSQL_DATABASE_NAME`)

# Requirements

 * You’ll need [`mgood/resolvable:latest`](https://github.com/mgood/resolvable) image running in order to resolve docker container names as valid domain names. This way it’s easy for your mysql client to connect, just point it to: `mysql.take-over-the-world.docker`.
 * Only short dokku names will work, so use `whatever.take-over-the-world` and the plugin will recognize it as `take-over-the-world`-type app, and then append `VHOST` at the end for the url.

# Configuration

Add those to your `.rc` file in the plugin root directory.

## Domain suffix

Add this suffix to container name to get a resolvable domain

```
DOMAIN_SUFFIX=".docker"
```

## Backup directory

Source directory of mysql backup files

```
BACKUP_DIR=${BACKUP_DIR:-$(dirname $0)/backup}
```


## Mysql root password

```
MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD:-""}
```

## Docker mysql image name

```
DOCKER_MYSQL_IMAGE=${DOCKER_MYSQL_IMAGE:-"mysql"}
```

## Email domains whitelist

Don’t anonymize those domain names:

```
EMAIL_DOMAIN_WHITELIST=(example.com whatever.rocks)
```
