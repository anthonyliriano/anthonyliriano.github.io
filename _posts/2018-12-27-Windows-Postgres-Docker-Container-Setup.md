---
layout: post
title: Postgres Docker Container on Windows (Work In Progress)
tags: [postgres, docker, windows]
comment: true
---


## Goal

Deploy a postgres container on a windows 10 and then connect via pgAdmin web interface.
[Postgres Container](https://hub.docker.com/_/postgres/)


### Switch to Linux Containers
After hours of troubleshooting, I realized that I was running under windows container which caused 'could not fsync file' and 'permission denied' error messages. Save yourself the headache -  ***Make sure you're running linux containers to succesfully deploy postgres*** To switch over right click the _Docker Desktop Icon Tray > Switch to Linux containers..._

_A Subtle reminder to add an image here_


### Create the Volume
```
docker volume create postgresql-volume
```


### Pull the Image and Run the container via docker-compose

Run the command below in the directory where the docker-compose.yml file exists. You'll be running detached, so you'll want to verify the container is running before attempting to connect. 
```docker-compose up -d```

```
version: '2.1'

services:

  postgres:
    image: postgres
    volumes:
      -  postgresql-volume:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=YOUR_PASSWORD
      - POSTGRES_USER=YOUR_USERNAME
      - POSTGRES_DB=YOUR_DATABASE
    ports:
      - "5432:5432"

volumes:
    postgresql-volume:
      external: true
```

### Pull & Run the PgAdmin4 Image

```docker run -p 80:80 -e "PGADMIN_DEFAULT_EMAIL=user@domain.com" -e "PGADMIN_DEFAULT_PASSWORD=SuperSecret" -d dpage/pgadmin4```


## Using psql within the container

Run a bash 
```docker exec -it NAME_OF_RUNNING_CONTAINER bash```

Run plsql
```psql -U POSTGRES_USER POSTGRES_DB```


### Get the Errror message below by running 

```docker logs postgres3```

### Output

```
2018-12-27 19:50:01.643 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
2018-12-27 19:50:01.643 UTC [1] LOG:  listening on IPv6 address "::", port 5432
2018-12-27 19:50:01.654 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
2018-12-27 19:50:13.799 UTC [54] LOG:  database system shutdown was interrupted; last known up at 2018-12-27 19:50:01 UTC
2018-12-27 19:50:18.195 UTC [54] LOG:  could not fsync file "./base/1": Permission denied
2018-12-27 19:50:19.124 UTC [54] LOG:  could not fsync file "./base/13066": Permission denied
2018-12-27 19:50:20.357 UTC [54] LOG:  could not fsync file "./base/13067": Permission denied
2018-12-27 19:50:20.360 UTC [54] LOG:  could not fsync file "./base": Permission denied
2018-12-27 19:50:20.549 UTC [54] LOG:  could not fsync file "./global": Permission denied
2018-12-27 19:50:20.555 UTC [54] LOG:  could not fsync file "./pg_commit_ts": Permission denied
2018-12-27 19:50:20.561 UTC [54] LOG:  could not fsync file "./pg_dynshmem": Permission denied
2018-12-27 19:50:20.578 UTC [54] LOG:  could not fsync file "./pg_logical/mappings": Permission denied
2018-12-27 19:50:20.586 UTC [54] LOG:  could not fsync file "./pg_logical/snapshots": Permission denied
2018-12-27 19:50:20.589 UTC [54] LOG:  could not fsync file "./pg_logical": Permission denied
2018-12-27 19:50:20.601 UTC [54] LOG:  could not fsync file "./pg_multixact/members": Permission denied
2018-12-27 19:50:20.611 UTC [54] LOG:  could not fsync file "./pg_multixact/offsets": Permission denied
2018-12-27 19:50:20.614 UTC [54] LOG:  could not fsync file "./pg_multixact": Permission denied
2018-12-27 19:50:20.620 UTC [54] LOG:  could not fsync file "./pg_notify": Permission denied
2018-12-27 19:50:20.624 UTC [54] LOG:  could not fsync file "./pg_replslot": Permission denied
2018-12-27 19:50:20.628 UTC [54] LOG:  could not fsync file "./pg_serial": Permission denied
2018-12-27 19:50:20.632 UTC [54] LOG:  could not fsync file "./pg_snapshots": Permission denied
2018-12-27 19:50:20.640 UTC [54] LOG:  could not fsync file "./pg_stat": Permission denied
2018-12-27 19:50:20.643 UTC [54] LOG:  could not fsync file "./pg_stat_tmp": Permission denied
2018-12-27 19:50:20.650 UTC [54] LOG:  could not fsync file "./pg_subtrans": Permission denied
2018-12-27 19:50:20.654 UTC [54] LOG:  could not fsync file "./pg_tblspc": Permission denied
2018-12-27 19:50:20.657 UTC [54] LOG:  could not fsync file "./pg_twophase": Permission denied
2018-12-27 19:50:20.669 UTC [54] LOG:  could not fsync file "./pg_wal/archive_status": Permission denied
2018-12-27 19:50:20.671 UTC [54] LOG:  could not fsync file "./pg_wal": Permission denied
2018-12-27 19:50:20.677 UTC [54] LOG:  could not fsync file "./pg_xact": Permission denied
2018-12-27 19:50:20.689 UTC [54] LOG:  could not fsync file ".": Permission denied
2018-12-27 19:50:20.691 UTC [54] LOG:  could not fsync file "pg_tblspc": Permission denied
2018-12-27 19:50:20.711 UTC [54] LOG:  database system was not properly shut down; automatic recovery in progress
2018-12-27 19:50:20.747 UTC [54] LOG:  invalid record length at 0/1650D90: wanted 24, got 0
2018-12-27 19:50:20.747 UTC [54] LOG:  redo is not required
2018-12-27 19:50:20.759 UTC [54] FATAL:  could not fsync file "base/1": Permission denied
2018-12-27 19:50:20.761 UTC [1] LOG:  startup process (PID 54) exited with exit code 1
2018-12-27 19:50:20.761 UTC [1] LOG:  aborting startup due to startup process failure
2018-12-27 19:50:20.764 UTC [1] LOG:  database system is shut down
```