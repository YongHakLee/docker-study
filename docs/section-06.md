# Creating a Production-Grade Workflow

## Development Workflow

Development -> Testing -> Deployment -> Development ...

## Flow Specifics

You -> feature (Pull Request) -> main -> Travis CI (Test) -> AWS Hosting

## Necessary Commands

- `npm run start`: Starts up a development server. _For development use only_
- `npm run test`: Runs test associated with the project
- `npm run build`: Build a **production** version of the application

## Creating the Dev Dockerfile

- `Dockerfile.dev`: use for dev

```dockerfile
# Dockerfile.dev
FROM node:lts-alpine

WORKDIR '/app'

COPY package.json .
RUN npm install

COPY . .

CMD ["npm", "run", "start"]
```

- `docker build -f Dockerfile.dev .`

## Starting The Container

- `docker run -p 3000:3000 IMAGE_ID`

## Docker Volumes

```shell
docker run -p 3000:3000 -v /app/node_modules -v $(pwd):/app <image_id>
```

- `-v /app/node_modules`: Put a bookmark on the node_modules folder
  - Without this: Error because we've deleted `node_modules` before.
  - We're saying this folder set in stone, don't try to mess with it. Don't try to map it up against anything else.
- `-v $(pwd):/app`: Map the pwd into the '/app' folder

## Shorthand with Docker Compose

```yml
# docker-compose.yml

services:
  web:
    build: .
    ports:
      - "3000:3000"
    volumes:
      - /app/node_modules
      - .:/app
```

## Overriding Dockerfile Selection

```yml
# docker-compose.yml

services:
  web:
    build:
      context: . # Option 1
      dockerfile: Dockerfile.dev # Option 2
    ports:
      - "3000:3000"
    volumes:
      - /app/node_modules
      - .:/app
```

## Do We Need Copy?

- Yes, for future...

## Executing Tests

```shell
docker run IMAGE_ID npm run test
```

- for see inside of it

```shell
docker run -it IMAGE_ID npm run test
```

## Live Updating Tests

```shell
docker compose up
docker exec -it CONTAINER_ID npm run test
```

## Docker Compose for Running Tests

```yml
# docker-compose.yml
services:
  web:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - /app/node_modules
      - .:/app
  tests:
    build:
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - /app/node_modules
      - .:/app
    command: ["npm", "run", "test"]
```

- We cannot interact with Tests.

## Shortcomings on Testing
