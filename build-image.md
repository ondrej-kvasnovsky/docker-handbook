# Build Image

Lets build and run a simple image. First we create a `Dockerfile` file that will serve as a definition of our image. We are using [hello-world](https://github.com/docker-library/hello-world) image as our base image.

```
FROM hello-world

MAINTAINER Ondrej Kvasnovsky <ondrej.kvasnovsky@gmail.com>
```

Now we can build the image.

```
docker build -f Dockerfile -t demo/hello-world .
```

We must be able to see the new image in the list of available images. 

```
➜  demo docker images
REPOSITORY                                           TAG                   IMAGE ID            CREATED             SIZE
demo/hello-world                                     latest                095848a4f47f        3 minutes ago       1.85kB
```

Once the image is available, we can run the image and that way, create a container. A container is a running instance of an image.

```
➜ demo docker run demo/hello-world

Hello from Docker!
...
```



