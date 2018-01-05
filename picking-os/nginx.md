# Nginx

Is an asynchronous web server  that is focused on concurrency and performance. For example, Nginx is sitting in-front of our application and when user requests an image, it redirects the request to server with static resources. Which means, it does not bother our application at all.

Lets create nginx docker container.

Docker file.

```
FROM nginx:1.9
MAINTAINER Ondrej Kvasnovsky <ondrej.kvasnovsky@gmail.com>

RUN rm /usr/share/nginx/html/*

COPY configs/nginx.conf /etc/nginx/nginx.conf
COPY configs/default.conf /etc/nginx/conf.d/default.conf

COPY certs/productionexample.crt /etc/ssl/certs/productionexample.crt
COPY certs/productionexample.key /etc/ssl/private/productionexample.key
COPY certs/dhparam.pem /etc/ssl/private/dhparam.pem

COPY docker-entrypoint /
RUN chmod +x /docker-entrypoint
ENTRYPOINT ["/docker-entrypoint"]

CMD ["nginx", "-g", "daemon off;"]
```

We can get all the files [here](https://www.dropbox.com/sh/5l400rrycpe81m5/AACRcys5LusPrgYJchdvKWWla?dl=0&preview=nginx+-+Customizing+the+official+nginx+image.zip).

`configs/nginx.conf` provides basic configuration of nginx.

```
# Number of workers to run, usually equals number of CPU cores.
worker_processes auto;

# Maximum number of opened files per process.
worker_rlimit_nofile 4096;

events {
  # Maximum number of simultaneous connections that can be opened by a worker process.
  worker_connections 1024;
}

http {
  include /etc/nginx/mime.types;
  default_type application/octet-stream;

  # ---------------------------------------------------------------------------
  # Security settings from:
  # https://gist.github.com/plentz/6737338

  # Disable nginx version from being displayed on errors.
  server_tokens off;

  # config to don't allow the browser to render the page inside an frame or iframe
  # and avoid clickjacking http://en.wikipedia.org/wiki/Clickjacking
  # if you need to allow [i]frames, you can use SAMEORIGIN or even set an uri with ALLOW-FROM uri
  # https://developer.mozilla.org/en-US/docs/HTTP/X-Frame-Options
  add_header X-Frame-Options SAMEORIGIN;

  # when serving user-supplied content, include a X-Content-Type-Options: nosniff header along with the Content-Type: header,
  # to disable content-type sniffing on some browsers.
  # https://www.owasp.org/index.php/List_of_useful_HTTP_headers
  # currently suppoorted in IE > 8 http://blogs.msdn.com/b/ie/archive/2008/09/02/ie8-security-part-vi-beta-2-update.aspx
  # http://msdn.microsoft.com/en-us/library/ie/gg622941(v=vs.85).aspx
  # 'soon' on Firefox https://bugzilla.mozilla.org/show_bug.cgi?id=471020
  add_header X-Content-Type-Options nosniff;

  # This header enables the Cross-site scripting (XSS) filter built into most recent web browsers.
  # It's usually enabled by default anyway, so the role of this header is to re-enable the filter for
  # this particular website if it was disabled by the user.
  # https://www.owasp.org/index.php/List_of_useful_HTTP_headers
  add_header X-XSS-Protection "1; mode=block";
  # ---------------------------------------------------------------------------

  # Avoid situations where a hostname is too long when dealing with vhosts.
  server_names_hash_bucket_size 64;
  server_names_hash_max_size 512;

  # Performance optimizations.
  sendfile on;
  tcp_nopush on;

  # http://nginx.org/en/docs/hash.html
  types_hash_max_size 2048;

  # Enable gzip for everything but IE6.
  gzip on;
  gzip_disable "msie6";

  # Default config for the app backend.
  include /etc/nginx/conf.d/default.conf;
}
```

`configs/default.conf` makes sure that requests are properly redirected.

```
upstream mobydock {
  # The web application.
  server mobydock:8000;
}

# In case you want 'www' addresses to be automatically redirected without 'www'.
server {
  listen 80;
  listen 443;
  server_name www.192.168.1.199;
  return 301 https://192.168.1.99$request_uri;
}

server {
  listen 80 default deferred;
  server_name 192.168.1.199;

  # All http traffic will get redirected to SSL.
  return 307 https://$host$request_uri;
}

server {
  # "deferred" reduces the number of formalities between the server and client.
  listen 443 default deferred;
  server_name 192.168.1.199;

  # Static asset path, which is read from the mobydock container's VOLUME.
  root /mobydock/static;

  # Ensure timeouts are equal across browsers and raise the max content-length size.
  keepalive_timeout 60;
  client_max_body_size 5m;

  # SSL goodness.
  ssl                       on;
  ssl_certificate           /etc/ssl/certs/productionexample.crt;
  ssl_certificate_key       /etc/ssl/private/productionexample.key;
  ssl_session_cache         shared:SSL:50m;
  ssl_session_timeout       5m;
  ssl_protocols             TLSv1 TLSv1.1 TLSv1.2;
  ssl_prefer_server_ciphers on;
  ssl_ciphers               "ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA";
  ssl_dhparam               /etc/ssl/private/dhparam.pem;
  ssl_ecdh_curve            secp384r1;
  add_header                Strict-Transport-Security 'max-age=63072000; includeSubDomains;' always;

  # Disallow access to hidden files and directories.
  location ~ /\. {
    return 404;
    access_log off;
    log_not_found off;
  }

  # Allow optionally writing an index.html file to take precedence over the upstream.
  try_files $uri $uri/index.html $uri.html @mobydock;

  # Attempt to load the favicon or fall back to status code 204.
  location = /favicon.ico {
    try_files /favicon.ico = 204;
    access_log off;
    log_not_found off;
  }

  # Load the web app back end with proper headers.
  location @mobydock {
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_redirect off;
    proxy_pass http://mobydock;
  }
}
```

`docker-entrypoint` when we run this, it will make sure we get proper nginx configuration for other environments.

```
#!/usr/bin/env bash
set -e

# This allows us to use the same template
# for development, staging and production.
CONFIG_PATH="/etc/nginx/conf.d/default.conf"
STAGING_IP="192.168.1.199"
STAGING_HOSTNAME="stagingserver"
DOMAIN_NAME="productionexample.com"

if [[ $(hostname) != "${STAGING_HOSTNAME}" ]]; then
  sed -i "s/${STAGING_IP}/${DOMAIN_NAME}/g" "${CONFIG_PATH}"
fi

# Execute the CMD from the Dockerfile.
exec "$@"
```

### Generate self signed certificates

Here is a sample how we can create self-signed certificates.

```
$ mkdir certs
$ openssl req -newkey rsa:2048 -nodes -sha256 -keyout certs/productionexample.key -x509 -days 3650 -out certs/productionexample.crt -subj "/C=US/ST=NewYourk/L=NewYourk/O=IT/CN=fakemobydock.com"
Generating a 2048 bit RSA private key
........+++
.................................................+++
writing new private key to 'certs/productionexample.key'
-----
```

[Diffie Hellman](https://en.wikipedia.org/wiki/Diffie–Hellman_key_exchange) key.

```
$ openssl dhparam -out certs/dhparam.pem 2048
$ cat certs/dhparam.pem
-----BEGIN DH PARAMETERS-----
MIIBCAKCAQEAqTwG+67Ja/d2Bv2T4YQa9N+3Dx84A/geGF4t2IYIH2spAbqVf5I/
XMZXBCOfxbJmjGOVVLYAkL9hQ6cssFUQWupuSOtDcLuMA9sx1bEn0Sa+HpkuLnzY
c1OxL0nJjX8tPqUM/djJCSr9KqUsRdDVWu5rcr1IFxmvlp9I1YWhqFmaLAyvB4QB
S/skk8fXgjTkoRuRic8WCIg6qlakWt/DVvm/Nja2PZnl3YKJROUch4e3Tdj1UJm+
oV06Na3E8092NMrqbn8bUVsI0TMzTRKOQVM7JbE6WD7OMQ2c2t8bbOZT4M9AZMbA
F60DS/3Yp+mmTXlfCNohg4olmr7OebnLGwIBAg==
-----END DH PARAMETERS-----
```



