- Listing Containers

```shell
docker ps
docker ps --all
```

- Container Lifecyle

`docker run = docker create + docker start`

- Restarting Stopped Containers

```shell
docker start ID
```

- Removing Stopped Containers

```shell
docker system prune
```

- Retrieving Log Outputs

```shell
docker logs ID
```

- Stopping Containers

```shell
docker stop ID
# SIGTERM, 10 Seconds
```

```shell
docker kill ID
# SIGKILL
```
