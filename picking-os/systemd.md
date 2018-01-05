# systemd

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



