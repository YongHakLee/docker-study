# 03. Building Custom Images Through Docker Server

## Creating Docker Images

- Dockerfile -> Docker client -> Docker Server -> Usable Image!
- Dockerfile: Configuration to define how our container should behave
  - Specify a base image -> Run some commands to install additional programs -> Specify a command to run on container startup

## Dockerfile Teardown

```shell
# Step 1: Use an existing docker image as a base

FROM alpine

# Step 2: Download and install a dependency

RUN apk add --update redis

# Step 3: Tell the image what to do when it starts as a container

CMD ["redis-server"]
```

- `FROM`, `RUN`, `CMD`: Instruction telling Docker Server what to do
- `alpine`, `apk ...`, `["redis...]`: Argument to the Instruction
- `alpine`: a base image
- `apk`: a package manager program

## The Build Process in Detail

- `docker build .`
- `FROM` -> image1 (File System snapshot) -> `RUN` runs a container using image1 -> image2 (FS snapshot) -> `CMD` runs a container using image2 -> image3

## Quiz

- Q1: What does the `FROM` command do?
  - A1: `FROM` copies the filesystem snapshot and default command from another image into the custom image we are building.
- Q2: The `hello-world` image has a filesystem snapshot has exactly _one file_ inside of it, the `hello` file. This is a program that is executed when you first execute the `hello-world` image as a container. The `hello-world` image has _absolutely no other programs inside of it_. With that in mind, what would happen if we tried to build an image with this Dockerfile:

```shell
FROM hello-world
RUN apk add nodejs
CMD ["node", "-e", "console.log('hi there');"]
```

- A2: We would get an error message during the `RUN apk add nodejs` command. We would see this error message because our image doesn't have an `apk` program, since it didn't inherit `apk` from `hello-world`.

- Q3: Why do we use `alpine` as a base image when building our own custom images?
  - A3: Alpine includes a default set of programs that are useful for setting up our own custom image. And Alpine is a very small image. This means that Docker can create containers out of our base image slightly faster.

- Q4: Are all custom images required to use `alpine` as a base image?
  - A4: No.

## Rebulids with Cache

```shell
FROM alpine

# Step 2: Download and install dependency


RUN apk add --update redis
# Using Cache

RUN apk add --update gcc

# Step 3: Tell the image what to do when it starts as container

CMD ["redis-server"]
```

- Ordering the commands is related with the Cache.

## Tagging an Image

```shell
docker build -t feint225/redis:latest .
```

- `feint225/redis:latest`: Tags the image
  - `DOCKER_ID/REPO_NAME:VERSION`
- `.`: Specifies the directory of files/folders to use for the build

```shell
docker run feint225/redis
```

## Quiz

### Q1

You are working on Python project, writing code to work with _only Python v3.8._ You write a Dockerfile like the following to run your code:

```shell
FROM python
RUN ["python", "main.py"]
```

You build your image, create a container from it, and everything works!

Then, _three years in the future_, you make a change to this project and rebuild the image. When you try to create a container, you get an error message!

**What is one possible reason to explain the error message you see?**

### A1

We didn't specify a version of the `python` image to use, so Docker automatically used the `latest` tag. That means we might have got Python v3.8 during the initial build, but maybe Python v4.5 (or some future version) when we rebuild the image three years later.

### Q2

If you ran the command `docker build . -t app1` how would you run the image that gets created?

### A2

`docker run app1`

### Q3

After running the command `docker build .` you see the following output:

```shell
=> => exporting layers                              0.0s
=> => writing image sha256:9dfadec01fefd446b8a918b  0.0s
```

How would you tag this image with a tag of `app1`?

### A3

`docker tag 9dfa app1`

### Q4

Which of the following `tag` commands correctly follows naming conventions?

### A4

`docker tag ece24c dockeruser/my-fancy-image`

### Q5

You decide to use an image created by another engineer with the following name:

`dockeruser/webapp:1.4.3-alpine3.10`

Which of the following is true?

### A5

This is image version 1.4.3. It likely used a base image of Alpine v3.10
