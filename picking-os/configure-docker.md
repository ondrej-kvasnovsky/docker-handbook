### Configure Docker

In case Docker is not installed on the server, follow the next steps. 

##### Install Docker

Ssh to the server.

```
$ ssh ondrej@192.168.1.199
```

Then this or follow [instructions](https://docs.docker.com/engine/installation/linux/docker-ce/debian/#set-up-the-repository) on docker web. 

```
$ sudo apt-get update
$ sudo apt-get install \
     apt-transport-https \
     ca-certificates \
     curl \
     gnupg2 \
     software-properties-common
$ curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg | sudo apt-key add -
$ sudo apt-key fingerprint 0EBFCD88
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
   $(lsb_release -cs) \
   stable"
$ sudo apt-get update
$ sudo apt-get install docker-ce
```

Verify the docker is installed. 

```
sudo docker run hello-world
```

##### Configure Docker

Run docker without root.

```
sudo usermod -aG docker ondrej
```

We can verify we can run docker without `sudo` now.

```
docker --version
```



