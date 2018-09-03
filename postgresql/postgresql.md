# Exploring PostgreSQL

This tutorial shows how you can use Docker to explore PostgreSQL. You can run the commands with Docker installed, or Docker for Windows in Linux mode. But you can also use [Play with Docker](https://labs.play-with-docker.com) to try this out.

### Start a new container running PostgreSQL
Here we're giving it a name (`postgres1`) and exposing port 5432 (the PostgreSQL default). We're running detached (`-d`)

```
docker run -d -p 5432:5432 --name postgres1 postgres
```

Check it's running with

```
docker ps
```

And view the log output with 

```
docker logs postgres1
```

### Create a database

We'll create a database and do it by creating a new interactive shell running on our `postgres1` container:

```
docker exec -it postgres1 sh
```

Create a new database
```
# createdb -U postgres mydb
```

Launch the `psql` utility connected to that database:

```
# psql -U postgres mydb
```

### Explore the databae

Now inside `psql`, let's run some basic commands. `\l` lists the databases. We'll also ask for the database version, the current date

```
mydb=# \l
mydb=# select version();
mydb=# select current_date;
mydb=# select 2+2;
```

Now let's do something a bit more interesting. We'll create a table:

```
mydb=# CREATE TABLE people (
mydb(#    id int,
mydb(# name varchar(80)
mydb(# );
CREATE TABLE
```

Then we'll insert a row into the table:

```
mydb=# INSERT INTO people (id,name) VALUES (1, 'Mark');
INSERT 0 1
```

And finally, check it's there

```
mydb=# SELECT * FROM people;
 id | name 
----+------
  1 | Mark
(1 row)
```

Now we can quit from `psql` and exit from our shell
```
mydb=# \q 
# exit
```

Of course our container is still running.

### stop and restart the container

Let's prove that we don't lose the data if we stop and restart the container.

```
docker stop postgres1
docker start postgres1
```

And rather than connect again to this container, let's test from another linked container.

```
docker run -it --rm --link postgres1:pg --name client1 postgres sh
```

Launch `psql` but connect to the other container (`-h pg`):
```
# psql -U postgres -h pg mydb
```

Now we should be able to access the same data:
```
mydb=# SELECT * FROM people;
 id | name 
----+------
  1 | Mark
(1 row)
```

Now we can quit from `psql` and exit from our shell, which will remove the `client1` container.

```
mydb=# \q 
# exit
```

And let's stop and remove the `postgres1` container with a single command (`-f` to force it to remove a running container)

```
docker rm -f postgres1
```

### Store data in a volume

Let's create another PostgreSQL container, but this time we'll ask it to store all data in a volume called `postgres-data`. We know that PostgreSQL stores its data in `/var/lib/postgresql/data`.

```
docker run -d -p 5432:5432 -v postgres-data:/var/lib/postgresql/data --name postgres1 postgres
```

Now let's attach a shell, create a database, a table and add one row:

```
docker exec -it postgres1 sh
# createdb -U postgres voldb
# psql -U postgres voldb
voldb=# CREATE TABLE people (id int, name varchar(80));
CREATE TABLE
voldb=# INSERT INTO people (id,name) VALUES (2, 'Steph');
INSERT 0 1
```

And exit out of `psql` and the shell:

```
voldb=# \q
# exit
```

We can find out information about the volume that we've created with `docker volume inspect`, including where on our local disk the data in that volume is being stored. Here's

```
$ docker volume inspect postgres-data
[
    {
        "CreatedAt": "2018-09-03T19:50:23Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/postgres-data/_data",
        "Name": "postgres-data",
        "Options": null,
        "Scope": "local"
    }
]
```

And if we take a look inside the local folder, we can see all the data that has been stored in that volume.

```
$ ls /var/lib/docker/volumes/postgres-data/_data/
PG_VERSION            pg_multixact          pg_tblspc
base                  pg_notify             pg_twophase
global                pg_replslot           pg_wal
pg_commit_ts          pg_serial             pg_xact
pg_dynshmem           pg_snapshots          postgresql.auto.conf
pg_hba.conf           pg_stat               postgresql.conf
pg_ident.conf         pg_stat_tmp           postmaster.opts
pg_logical            pg_subtrans           postmaster.pid
```

Let's stop and remove this container now. Unlike last time we did this, we won't lose the database we created, since it's in a volume that hasn't been deleted

```
docker rm -f postgres1
```

### Attach an existing volume to a new container

Let's now start up a brand new container called `postgres2` but attach the existing `postgres-data` volume that contains our database (called `voldb`):

```
docker run -d -p 5432:5432 -v postgres-data:/var/lib/postgresql/data --name postgres2 postgres
```

Once that starts up, let's run a `psql` session inside and check that the database, table and data are still all present and correct:

```
docker exec -it postgres2 sh
# psql -U postgres voldb
voldb=# SELECT * FROM people;
 id | name
----+-------
  2 | Steph
(1 row)
```

And exit out again:
```
voldb=# \q
# exit
```

And now, let's really clean up. Not only will we remove the second container, but we'll then remove the `postgres-data` volume. So now the contents of the database are deleted as well.

```
docker rm -f postgres2
docker volume rm postgres-data
```