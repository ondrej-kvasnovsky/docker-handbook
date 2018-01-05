### Debian

Lets configure Debian 9 OS to run our Docker containers. We are going to use it as our staging server.

First [download](https://www.debian.org/distrib/netinst) Debian distribution. Then install that into your virtual box. Just ssh and headless mode will be fine. Create at least 4GB disc.

Then we can do basic configuration.

```
$ su
$ apt-get install sudo
$ adduser ondrej sudo
$ sudo vi /etc/sudoers
```

And change this line to, that will stop requiring password for users who are in sudo group.

```
%sudo     ALL=(ALL:ALL) NOPASSWD:ALL
```

Then logout from root and your session and login again. We should see the difference when we run.

```
$ whoami
ondrej
$ sudo whoami
root
```

Lets setup static IP address to avoid changing IP every time when we start that virtual machine.

```
$ ip address
$ sudo vi /ect/network/interfaces
```

Make the file look like this.

```
source /etc/network/interfaces.d/*

auto lo
iface lo inet loopback

allow-hotplug enp0s3
iface eth0 inet static
  address 192.168.199
  netmask 255.255.255.0
  gateway 192.168.1.1
```

Then.

```
sudo reboot --reboot
```

Then verify the static address is assigned.

```
ip address
```



