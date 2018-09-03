# Exploring Redis

This tutorial shows how you can use Docker to explore Redis. You can run the commands with Docker installed, or Docker for Windows in Linux mode. But you can also use [Play with Docker](https://labs.play-with-docker.com) to try this out.

### Start a new container running Redis
Here we're giving it a name (redis1) and exposing port 6379 (the redis default)

```
docker run -d -p 6379:6379 --name redis1 redis
```

Check it's running with

```
docker ps
```

And view the log output with 

```
docker logs redis1
```

### Run the redis-cli in the container

We're going to start a new interactive session (-it) inside the running container, and use it to run `redis-cli`. We could run it directly, but for now, let's just start a shell with `sh`:

```
docker exec -it redis1 sh
```

And now we're attached to our container. Let's run `redis-cli`:

```
# redis-cli
```

### Try out some redis commands

Send a "ping", should get "PONG" back
```
127.0.0.1:6379> ping
PONG
```

Try out some more commands (set a key and increment a counter)
```
127.0.0.1:6379> set name mark
OK
127.0.0.1:6379> get name
"mark"
127.0.0.1:6379> incr counter
(integer) 1
127.0.0.1:6379> incr counter
(integer) 2
127.0.0.1:6379> get counter
"2"
```

And when we're done exit out of `redis-cli` and `sh`:
```
127.0.0.1:6379> exit
# exit
```

### Connect from another linked container

Now let's start another container, called `client1`. We'll base it of the `redis` image but we're only using it to run the `redis-cli` so we'll just aski it to run `sh` in interactive mode `-it`. This means it won't be running redis itself. We specify `--rm` so it will delete itself after the shell exits.

We link it to the `redis1` container (which is still running), and it will be referred to from within this container simply as `redis`.

```
docker run -it --rm --link redis1:redis --name client1 redis sh
```

Now in this container let's start `redis-cli` and connect to `redis1` which has the name `redis`:

```
# redis-cli -h redis
redis:6379>
```

And now let's issue some commands. Since we didn't stop the original `redis1` container, the keys we created earlier should still be there:

```
redis:6379> get name
"mark"
redis:6379> get counter
"2"
```

And now exit out of `redis-cli` and `sh`:
```
redis:6379> exit
# exit
```
This has actually caused our `client1` container to delete itself. So even if we ask to see all containers including stopped ones with:

```
docker ps -a
```
We should only see our single Docker container.

### Clean up

Let's stop the `redis1` container and then delete it.
```
docker stop redis1
docker rm redis1
```

We have also downloaded the `redis` image, which we will see in the list
```
docker image ls
```
It's only 83MB, so we may want to keep it for next time we run a docker image, but if we're sure we don't need it anymore, we can remove the image to free up disk space and get us back to exactly where we started before:

```
docker image rm redis
```
