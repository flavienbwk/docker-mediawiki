# Semantic MediaWiki on Docker

This repo contains an opinionated installation of MediaWiki. It has all the semantic wiki extensions installed and configured and is configured with visualeditor support (aka the works).

## Installation

```bash
docker-compose build
```

## Configuration

```bash
cp .env.example .env
# EDIT YOUR .env !!!
```

## Startup

```bash
docker-compose up -d
```

You need to run the migrations after the initial start of the containers so the tables needed by the semantic extensions are installed.

> You can also choose to import old data (see below) and run the migration after that.
> 
> ```bash
> docker-compose exec wiki php maintenance/update.php --skip-external-dependencies --quick
> ```

## Backup database and images

```bash
docker-compose exec mysql mysqldump -uroot -p$MYSQL_ROOT_PASSWORD mediawiki > backup.sql

# TODO image volume backup
```

## Import data from old wiki

You can SQL file containing a `mysqldump` in `./data.sql` and then run the following commands.

```bash
cat docker-composer.override.yml <<<EOD
mysql:
  volumes:
    - ./data.sql:/tmp/data.sql
EOD

docker-compose up -d

docker-compose exec mysql mysql -uroot -p$MYSQL_ROOT_PASSWORD mediawiki -e "source /tmp/data.sql"

docker-compose exec wiki php maintenance/update.php --skip-external-dependencies --quick
```

If you used `<code><pre>` instead of `<syntaxhighlight>` you can do this:

```bash
docker-compose exec wiki php extensions/ReplaceText/maintenance/replaceAll.php --nsall '<code><pre>' '<syntaxhighlight lang="bash">'
docker-compose exec wiki php extensions/ReplaceText/maintenance/replaceAll.php --nsall '</pre></code>' '</syntaxhighlight>'
```

Now is also the time to do some final cleanup and optimization after having migrated your wiki to this stack.

```bash
docker-compose exec wiki php maintenance/runJobs.php
```

If you had to wait a long time for the jobs to run you might want to run the updater again so it can optimize the db tables after the jobs have run.

```bash
docker-compose exec wiki php maintenance/update.php --skip-external-dependencies --quick
```
