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

### See how docker instances were started

```
$ docker inspect -f "{{.Name}} {{.Config.Cmd}}" $(docker ps -a -q)
/my-service [/bin/sh -c pm2-docker -i max app.js]
/logrotate [cron]
/logs [/bin/sh -c exec fluentd -c /fluentd/etc/${FLUENTD_CONF} -p /fluentd/plugins $FLUENTD_OPT]
```

Or more standard command that print out all data, without shortening it.

```
$ docker ps -a --no-trunc
```



