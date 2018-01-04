# Basic Commands

Here are couple of basic commands for using Docker.

> Docker containers are immutable.

### Pull a docker image from hub

```
docker pull busybox:latest
```

### Run Docker image to execute an operation

We can run docker image, execute an operation and kill the docker container. `--rm` flag will reference to the Docker container when it is finished.

The following command will start docker container, execute `echo` and exit.

```
docker run --rm busybox:latest /bin/echo "Hello World"
```

If we don't use `--rm` flag, the container is not removed and we can see it when we run `docker ps -a`. Then we need to remove it manually.

### Run Docker image

We can run docker image and connect to it. `-it` flag says Docker to run the container in interactive mode.

```
docker run -it --rm busybox:latest
```

### Manage Docker images

We list all the docker images.

```
docker images
```

Then we can remove Docker image.

```
docker rmi <id of docker image>
```

### Running Docker containers

First, we can list all running containers. `-a` flag will make sure we see all the containers.

```
docker ps -a
```

We can remove running Docker container.

```
docker rm <id of container or just couple of first letters from the id>
```



