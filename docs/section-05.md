# 05. Docker Compose with Multiple Local Containers

## App Overview

- Web shows 'Number of visits: 10'
  - Node App -> Redis (_Visits=10_)

- Seperate Docker Containers for the Node application (Multiple) and the Redis server (1).

## Assembling a Dockerfile

## Introducing Docker Compose

- Seperate CLI that gets installed along with Docker.
- Used to start up multiple Docker containers at the same time
- Automates some of the long-winded arguments we were passing to 'docker run'

## Docker Compose Files

```shell
docker build -t feint225/visits:latest .
docker run -p 8080:8080 feint225/visits
```

1. `docker-compose.yml` Contains all the options we'd normally pass to docker-cli
2. docker-compose CLI

## Docker Compose Commands

- `docker run myimage` -> `docker-compose up`
- `docker build .` + `docker run myimage` -> `docker-compose up --build`

## Stopping Docker Compose Containers

- `docker-compose up -d`: Launch in background
- `docker-compose down`: Stop Containers
