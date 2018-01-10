## Java Server

We are going to use an application that was created in [this article](https://www.gitbook.com/book/ondrej-kvasnovsky/spring-handbook/edit#/edit/master/chapter1/rest-api.md?_k=dalfz5). It is using Spring 5 and it implements a simple REST API controller `GET /hello` that returns `Hello` string as a reponse. Lets get [this jar file](https://www.dropbox.com/s/uk5qswtmnljry8y/demo-0.0.1-SNAPSHOT.jar?dl=0) in case we don't want to build the application now.

Lets create the following directory structure with all the required files before we start.

```
➜ mkdir -p hello-word/build/libs
➜ cd hello-world
➜ touch Dockerfile
➜ wget https://www.dropbox.com/s/uk5qswtmnljry8y/demo-0.0.1-SNAPSHOT.jar\?dl\=0 -O  build/libs/demo.jar
➜ tree
.
├── Dockerfile
└── build
    └── libs
        └── demo.jar
```

Now we want to create Dockerfile that will create an image with demo.jar. When we start that docker image, we want to start the Sprint REST API on port 8080. Study the [documentation](https://docs.docker.com/engine/reference/builder) what each docker instruction does.

```
FROM openjdk:8u151-jre-slim

WORKDIR /app

COPY build/libs/demo.jar .

EXPOSE 8080

CMD ["java", "-jar", "demo.jar"]
```

Now we can try to build.

```
➜ hello-world docker build -t spring/rest-api .
Sending build context to Docker daemon  15.85MB
Step 1/1 : FROM openjdk:8u151-jre-slim
 ---> 66d599bf7861
Successfully built 66d599bf7861
Successfully tagged spring/rest-api:latest
```

Once build is done, we can star the docker. Study the documentation to learn how [docker run](https://docs.docker.com/engine/reference/run/) works with all the options. Here we use `-d` flag to run container in detached mode. `-p` flag says port mapping, from docker container to our system.

```
➜ docker run -d -p 8080:8080 spring/rest-api
e13dfb803fb264fe61767f0378cda2029debad8488a597a22a9e6100fbcebd5e
```

First, check if our docker container is running.

```
➜ docker ps
CONTAINER ID        IMAGE               COMMAND                CREATED              STATUS              PORTS                    NAMES
e13dfb803fb2        spring/rest-api     "java -jar demo.jar"   About a minute ago   Up About a minute   0.0.0.0:8080->8080/tcp   nervous_ardinghelli
```

If the docker container is up, we can try to talk to the server which is started inside the docker container.

```
➜ curl localhost:8080/hello
Hello
```

The consequent question is how to stop the docker container we just started. We need do it by using container id, or at least the begging of the id string. `e13` should be enough to identify the docker container and stop it.

```
docker stop e13
```



