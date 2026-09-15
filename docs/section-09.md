# Dockerizing Multiple Services

## Dockerizing React App

```shell
docker build -f Dockerfile.dev .
```

```dockerfile
# client/Dockerfile.dev

FROM node:lts-alpine
WORKDIR '/app'
COPY ./package.json ./
RUN npm install
COPY . .
CMD ["npm", "run", "start"]
```

```shell
docker run CONTAINER_ID
```

## Dockerizing Generic Node Apps

```dockerfile
# server/Dockerfile.dev
# worker/Dockerfile.dev
FROM node:lts-alpine
WORKDIR "/app"
COPY ./package.json ./
RUN npm install
COPY . .
CMD ["npm", "run", "dev"]
```

- Test with a `docker run` command.

## Environment Variables with Docker Compose

- `variableName=value`: Sets a variable in the container at _run time_
- `variableName`: Sets a variable in the container at _run time_. Value is taken from _your computer_.

## Nginx Path Routing

- `/index.html` -> React Server
- `/main.js` -> React Server
- `/values/all` -> Express Server
- `/values/current` -> Express Server

- `Nginx`: What's the req start with?
  - `/`: -> React Server
  - `/api`: -> Express Server

## Routing with Nginx

- `default.conf`: Adds configuration rules to Nginx
  - Tell Nginx that there is an 'upstream' server at client:3000
  - Tell Nginx that there is an 'upstream' server at server:5000
  - Listen on port 80
  - If anyone comes to '/' send them to client upstream
  - If anyone comes to '/api' send them to server upstream
