# 04. Making Real Projects with Docker

## Project Outline

1. Create Node JS web app
2. Create a Dockerfile
3. Build image from dockerfile
4. Run image as container
5. Connect to web app from a browser

## A Few Planned Errors

### Node Js Apps

1. Have to install dependencies before running the app by `npm install`: Assumes 'npm' is installed!
2. Have to run a command to start up the server by `npm start`: Assumes 'npm' is installed!

## Base Image Issues

Navigate to `hub.docker.com` and explore to find an image.

## A Few Missing Files

FS Snapshot in an image doesn't have the files we need.

## Copying Build Files

```shell
COPY ./ ./
```

- the first `./`: Path to folder to copy from on **your machine** relative to build context
- the second `./`: Place to copy stuff to inside **the container**

## Container Port Mapping

```shell
docker run -p 8080:8080 <image name>
```

- the first `8080`: Route incoming requests to this port on local host to...
- the second `8080`: ...this port inside the container

## Quiz

### Q1

Your team's microservice listens on port 3000 inside the container. To access it on port 9000 on your host machine, which command is correct?

### A1

`docker run -p 9000:3000 myapp`

### Q2

If you have multiple containers that all need to expose port 80, which approach would work?

### A2

Map them to different host ports (8080:80, 8081:80, etc.)

### Q3

You have three microservices: frontend (port 3000), backend (port 5000), and database (port 5432). If all are running as separate containers on the same host, which statement is correct?

### A3

Each container has its own isolated network namespace.

## Specifying a Working Directory

```shell
WORKDIR /usr/app
```

- `/usr/app`: Any following command will be executed relative to this path in the container

## Unnecessary Rebuilds

- Just one file has changed.

## Minimizing Cache Busting and Rebuilds

```shell
COPY ./file_unchanged_often ./
RUN npm install
COPY ./ ./
```

- COPY the files required to RUN the command

## Quiz

### Q1

You are working on a Python project. The project directory has two files: `requirements.txt` and `main.py`.

Your Dockerfile to build the project looks like this:

```shell
FROM python
WORKDIR /app
ADD ./ ./
RUN pip install -r requirements.txt
CMD ["python", "main.py"]
```

The `requirements.txt` lists dependencies for the project. It does not change very often. It is used by a tool called `pip` to install dependencies into the project. This takes several minutes.

Your `main.py` file changes very often. Every time you change it, rebuilding your image takes several minutes!

**How can you change the Dockerfile to speed up the build process?**

### A1

Change the Dockerfile to:

```shell
FROM python
WORKDIR /app
ADD ./requirements.txt ./
RUN pip install -r requirements.txt
ADD ./main.py ./
CMD ["python", "main.py"]
```
