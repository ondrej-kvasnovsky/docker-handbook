# Running Services

We are not going to explain all about `systemd`, just what we are going to use to run our services as docker containers.

### Why systemd?

* concise "unit files"
* process dependencies
* unified toolset

### How to use systemd?

#### Unit files

List unit files created by Debian.

```
sudo systemctl list-unit-files
```

Unit files have multiple sections. Unit section have description and dependencies, like dependency to network.target that needs to be executed first.

Service sections have type `oneshot` and it makes sure that action will be executed.

Install sections says another dependency, when this should be started.

```
[Unit]
Description=Restore iptables firewall rules
Before=network.target
Conflicts=shutdown.target

[Service]
Type=oneshot
ExecStart=/sbin/iptables-restore /var/lib/iptables/rules-save

[Install]
WantedBy=basic.target
```

#### Postgres service

Service sections can have multiple phases. `%p` will resolve to name of the service, which is name of the file, `postgres.service`. It says the service needs to be started after docker service is up. Also, it says the service needs to be always restarted, which will make the service running all the time.

```
[Unit]
Description=Run %p
Requires=docker.service
After=docker.service

[Service]
Restart=always
ExecStartPre=-/usr/bin/docker kill %p
ExecStartPre=-/usr/bin/docker rm -f %p
ExecStart=/usr/bin/docker run --rm --name %p \
  -v /var/lib/postgresql/data:/var/lib/postgresql/data \
  -p 5432:5432 %p:9.4.5
ExecStop=/usr/bin/docker stop %p

[Install]
WantedBy=multi-user.target
```

#### Redis service

Here is the Redis service, `redis.service`.

```
[Unit]
Description=Run %p
Requires=docker.service
After=docker.service

[Service]
Restart=always
ExecStartPre=-/usr/bin/docker kill %p
ExecStartPre=-/usr/bin/docker rm -f %p
ExecStart=/usr/bin/docker run --rm --name %p \
  -v /var/lib/%p/data:/var/lib/%p/data -p 6379:6379 %p:2.8.22
ExecStop=/usr/bin/docker stop %p

[Install]
WantedBy=multi-user.target
```

### Put unit files on staging server

Lets see how we can put postgres.service on server and apply that.

First connect to the server and create the file. Then change owner.

```
$ ssh ondrej@192.168.1.199
$ sudo vi /etc/systemd/system/postgres.service
$ sudo chown ondrej:ondrej /etc/systemd/system/postgres.service
$ sudo vi /etc/systemd/system/redis.service
$ sudo chown ondrej:ondrej /etc/systemd/system/redis.service
```

Now, lets enable the services. Enable will make sure it is loaded at the system boot. Start will start it immediately.

```
$ sudo systemctl enable postgres.service
$ sudo systemctl start postgres.service
$ sudo systemctl enable redis.service
$ sudo systemctl start redis.service
```

Now we can try to play with the services. Lets try to restart redis service and see, that redis the docker container was just started.

```
$ sudo systemctl restart redis.service
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
95e593bbb735        redis:2.8.22        "/entrypoint.sh redi…"   1 second ago        Up 1 second         0.0.0.0:6379->6379/tcp   redis
0d859124baf9        postgres:9.4.5      "/docker-entrypoint.…"   6 minutes ago       Up 6 minutes        0.0.0.0:5432->5432/tcp   postgres
```

### Disable a service

We can disable the service.

```
$ sudo systemctl disable redis.service
```

### Service status

We can find out the status of the service. This is useful when debugging the service.

    $ sudo systemctl status redis.service
    ● redis.service - Run redis
       Loaded: loaded (/etc/systemd/system/redis.service; enabled; vendor preset: enabled)
       Active: active (running) since Thu 2018-01-04 22:29:36 PST; 4min 27s ago
      Process: 1513 ExecStop=/usr/bin/docker stop redis (code=exited, status=0/SUCCESS)
      Process: 1570 ExecStartPre=/usr/bin/docker rm -f redis (code=exited, status=1/FAILURE)
      Process: 1565 ExecStartPre=/usr/bin/docker kill redis (code=exited, status=1/FAILURE)
     Main PID: 1575 (docker)
        Tasks: 8 (limit: 4915)
       Memory: 4.6M
          CPU: 26ms
       CGroup: /system.slice/redis.service
               └─1575 /usr/bin/docker run --rm --name redis -v /var/lib/redis/data:/var/lib/redis/data -p 6379:6379 redis:2.8.22

    Jan 04 22:29:36 stagingserver docker[1575]:  |`-._`-._    `-.__.-'    _.-'_.-'|
    Jan 04 22:29:36 stagingserver docker[1575]:  |    `-._`-._        _.-'_.-'    |
    Jan 04 22:29:36 stagingserver docker[1575]:   `-._    `-._`-.__.-'_.-'    _.-'
    Jan 04 22:29:36 stagingserver docker[1575]:       `-._    `-.__.-'    _.-'
    Jan 04 22:29:36 stagingserver docker[1575]:           `-._        _.-'
    Jan 04 22:29:36 stagingserver docker[1575]:               `-.__.-'
    Jan 04 22:29:36 stagingserver docker[1575]: [1] 05 Jan 06:29:36.877 # Server started, Redis version 2.8.22
    Jan 04 22:29:36 stagingserver docker[1575]: [1] 05 Jan 06:29:36.877 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition.
    Jan 04 22:29:36 stagingserver docker[1575]: [1] 05 Jan 06:29:36.877 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somax
    Jan 04 22:29:36 stagingserver docker[1575]: [1] 05 Jan 06:29:36.877 * The server is now ready to accept connections on port 6379
    ...

### Reloading services

When we update the service file, we want to reload the changes. 

```
$ sudo systemctl daemon-reload
```

### Service crash

What happens when the service crashes? Lets try it out. Lets kill Redis service. We can see that when we kill the service, is is started again, thanks to `systemd`.

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
95e593bbb735        redis:2.8.22        "/entrypoint.sh redi…"   8 minutes ago       Up 8 minutes        0.0.0.0:6379->6379/tcp   redis
0d859124baf9        postgres:9.4.5      "/docker-entrypoint.…"   14 minutes ago      Up 14 minutes       0.0.0.0:5432->5432/tcp   postgres

$ docker stop 95e593bbb735
95e593bbb735

$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                  PORTS                    NAMES
0629c8d39127        redis:2.8.22        "/entrypoint.sh redi…"   2 seconds ago       Up Less than a second   0.0.0.0:6379->6379/tcp   redis
0d859124baf9        postgres:9.4.5      "/docker-entrypoint.…"   14 minutes ago      Up 14 minutes           0.0.0.0:5432->5432/tcp   postgres
```



