### Configure SSH

First generate key on host system.

```
$ ssh-keygen -t rsa -b 4096
```

Add the generated key, so it is going to be automatically picked.

```
$ ssh-add
```

Connect to the Debian server and copy the public key on the server as authorized key.

```
$ ssh ondrej@192.168.1.199 mkdir .ssh
$ cat id_rsa.pub | ssh ondrej@192.168.1.199 'cat >> .ssh/authorized_keys'
$ ssh ondrej@192.168.1.199 'chmod 700 .ssh; chmod 640 .ssh/authorized_keys'
```

Now we can connect via SSH, without password.

```
$ ssh ondrej@192.168.1.199
```

Disable password logins to make the server more secure.

```
$ sudo vi /etc/ssh/sshd_config
```

Then we update `PasswordAuthentication` from `yes` to `no`.

```
PasswordAuthentication no
```

Then we need to restart SSH to apply changes.

```
$ sudo systemctl restart ssh
```



