# Debugging

### AWS instances with docker are killed

When we want find out why an ec2 instance was killed, we need to first check if docker container is present. We need to use `-a` flag to tell docker to return also containers that are not running.

```
$ docker ps -a
CONTAINER ID        IMAGE          COMMAND                  CREATED             STATUS                     PORTS               NAMES
ba5925ec4a87        myservice...   "/bin/sh -c 'node ..."   6 minutes ago       Exited (1) 5 minutes ago                       myservice
```

Then we want to use name of the service and get logs.

```
$ docker logs myservice
...logs...
```



