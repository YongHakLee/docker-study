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

```shell
`docker build -f Dockerfile.dev .`
```

## Starting The Container

```shell
`docker run -p 3000:3000 IMAGE_ID`
```

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
      - .:/app # $(pwd):/app
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
      - .:/app # $(pwd):/app
```

## Do We Need Copy?

```shell
COPY . .
```

- Yes, for future...

## Executing Tests

```shell
docker run IMAGE_ID npm run test
```

- To see inside of it: `-it`

```shell
docker run -it IMAGE_ID npm run test
```

## Live Updating Tests

- Compare the number of 'Tests'

```js
// frontend/src/App.test.js

import { render, screen } from "@testing-library/react";
import App from "./App";

test("renders learn react link", () => {
  // test 1
  render(<App />);
  const linkElement = screen.getByText(/learn react/i);
  expect(linkElement).toBeInTheDocument();
});

test("renders learn react link", () => {
  // test 2
  render(<App />);
  const linkElement = screen.getByText(/learn react/i);
  expect(linkElement).toBeInTheDocument();
});
```

```shell
docker compose up
docker exec -it CONTAINER_ID npm run test # In the second terminal
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

- Test Container
  - `npm run test`
  - stdin / stdout / stderr

- Web Container
  - `npm run start`
  - stdin / stdout / stderr

- Our terminal connect to `stdin` of the primary process.

```shell
docker attach CONTAINER_ID
# enter q, p, ... not available
```

```shell
docker exec -it CONTAINER_ID sh
ps
```

- There are many processes.
- But our terminal connect to `stdin` of the primary process.

## Need for Nginx

- Dev Server -> Production Server (Nginx)

## Multi-Step Docker Builds

1. Use node:lts-alpine
2. Copy the package.json file
3. Install dependencies

- Deps only needed to execute 'npm run build'!

4. Run 'npm run build'
5. Start nginx

- Where's nginx from?

### Build Phase

1. Use node:lts-alpine
2. Copy the package.json file
3. Install dependencies
4. Run 'npm run build'

### Run Phase

1. Use nginx
2. Copy over the result of 'npm run build'
3. Start nginx

## Running Nginx

```dockerfile
# dockerfile
FROM node:lts-alpine AS builder

WORKDIR '/app'
COPY package.json .
RUN npm install
COPY . .
RUN npm run build

FROM nginx
COPY --from=builder /app/build /usr/share/nginx/html
```

```shell
docker build .
docker run -p 8080:80 IMAGE_ID
```
