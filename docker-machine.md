# Docker Machine

[Docker machine](https://docs.docker.com/machine/overview/) has two use cases:

* I have an older desktop system and want to run Docker on Mac or Windows
* I want to provision Docker hosts on remote systems

Difference between docker engine and docker machine is: 

* docker engine is what is running our applications, or actually doing the work
* docker machine is for managing docker hosts \(that are running docker engine\)

### Create new host

```
$ docker-machine create --driver virtualbox default
Creating CA: /Users/ondrej/.docker/machine/certs/ca.pem
Creating client certificate: /Users/ondrej/.docker/machine/certs/cert.pem
Running pre-create checks...
(default) Image cache directory does not exist, creating it at /Users/ondrej/.docker/machine/cache...
(default) No default Boot2Docker ISO found locally, downloading the latest release...
(default) Latest release for github.com/boot2docker/boot2docker is v18.01.0-ce
(default) Downloading /Users/ondrej/.docker/machine/cache/boot2docker.iso from https://github.com/boot2docker/boot2docker/releases/download/v18.01.0-ce/boot2docker.iso...
(default) 0%....10%....20%....30%....40%....50%....60%....70%....80%....90%....100%
Creating machine...
(default) Copying /Users/ondrej/.docker/machine/cache/boot2docker.iso to /Users/ondrej/.docker/machine/machines/default/boot2docker.iso...
(default) Creating VirtualBox VM...
(default) Creating SSH key...
(default) Starting the VM...
(default) Check network to re-create if needed...
(default) Found a new host-only adapter: "vboxnet0"
(default) Waiting for an IP...
Waiting for machine to be running, this may take a few minutes...
Detecting operating system of created instance...
Waiting for SSH to be available...
Detecting the provisioner...
Provisioning with boot2docker...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env default
```

### Get list of hosts

```
$ docker-machine ls
NAME      ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
default   -        virtualbox   Running   tcp://192.168.99.100:2376           v18.01.0-ce
```

### Get environment commands

```
$ docker-machine env default
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.100:2376"
export DOCKER_CERT_PATH="/Users/ondrej/.docker/machine/machines/default"
export DOCKER_MACHINE_NAME="default"
# Run this command to configure your shell:
# eval $(docker-machine env default)
```

Then we are supposed to run this to connect our shell to the docker host. 

```
$ eval "$(docker-machine env default)"
```

### Get IP address of docker host

```
$ docker-machine ip default
192.168.99.100
```

### Test docker machine

Get nginx docker and run it. Then try to talk to the nginx. 

```
$ docker run -d -p 8000:80 nginx
...
$ curl $(docker-machine ip default):8000
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

Try to kill docker machine, if you do that, no docker engines will be available. See what happends when you run `curl $(docker-machine ip default):8000` again. 

```
$ docker-machine stop default
$ docker-machine ls
NAME      ACTIVE   DRIVER       STATE     URL   SWARM   DOCKER    ERRORS
default   -        virtualbox   Stopped                 Unknown
$ docker-machine start default
```

### Provisioning on Cloud

Docker machine can provision docker engines in a cloud, see [documentation](https://docs.docker.com/machine/get-started-cloud/) for more. 

