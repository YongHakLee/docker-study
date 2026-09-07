# Step 1: Use an existing docker image as a base

FROM alpine

# Step 2: Download and install a dependency

RUN apk add --update redis

# Step 3: Tell the image what to do when it starts as a container

CMD ["redis-server"]