# Dockerized Web App

Lets make an application using [Flask](http://flask.pocoo.org) \(Python microframework\) and ship it in Docker container.

There will be the following folder structure and files.

```
.
└── mobydock
    ├── Dockerfile
    ├── config
    │   ├── __init__.py
    │   └── settings.py
    ├── docker-compose.yml
    ├── instance
    │   ├── __init__.py
    │   ├── settings.py
    │   └── settings.py.productionnple
    ├── requirements.txt
    └── mobydock
        ├── __init__.py
        ├── app.py
        ├── static
        │   ├── docker-logo.png.html
        │   └── main.css
        └── templates
            └── layout.html
```

### Dockerfile

Here is Docker file that will build image for our example.

```
FROM python:2.7-slim
MAINTAINER Ondrej Kvasnovsky <ondrej.kvasnovsky@gmail.com>

RUN mkdir /usr/share/man/man7
RUN mkdir /usr/share/man/man1
RUN apt-get update && apt-get install -qq -y build-essential libpq-dev postgresql-client-9.4 --fix-missing --no-install-recommends

ENV INSTALL_PATH /mobydock
RUN mkdir -p $INSTALL_PATH

WORKDIR $INSTALL_PATH

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

COPY . .

VOLUME ["static"]

CMD gunicorn -b 0.0.0.0:8000 "mobydock.app:create_app()"
```

### Docker Compose

For development, we can simplify development using `docker-compose`. It will prepare our, for example, local development environment.

We use three different images: postgres, redis and mobydock. Docker compose will make sure there is PostgreSQL and Redis databases. And it also maps our source code to docker so changes will be available immediately.

```
postgres:
  image: postgres:9.4.5
  environment:
    POSTGRES_USER: mobydock
    POSTGRES_PASSWORD: yourpassword
  ports:
    - '5432:5432'
  volumes:
    - ~/.docker-volumes/mobydock/postgresql/data:/var/lib/postgresql/data

redis:
  image: redis:2.8.22
  ports:
    - '6379:6379'
  volumes:
    - ~/.docker-volumes/mobydock/redis/data:/var/lib/redis/data

mobydock:
  build: .
  command: gunicorn -b 0.0.0.0:8000 --reload --access-logfile - "mobydock.app:create_app()"
  environment:
    PYTHONUNBUFFERED: "true"
  links:
    - postgres
    - redis
  volumes:
    - .:/mobydock
  ports:
    - '8000:8000'
```

> volumes will bind our local code to docker, so we will be able to see our changes immediately in docker container, that is very useful for local development

After we have the docker compose file, we can start it all up.

```
docker-compose up
```

### Other source code files

Other source code can be downloaded from [here](https://www.dropbox.com/sh/5l400rrycpe81m5/AACRcys5LusPrgYJchdvKWWla?dl=0).

### Connect to containers

We can see what containers were started. We can run `docker ps` or `docker-compose` ps in the project directory.

```
➜  mobydock docker-compose ps
       Name                      Command               State           Ports
-------------------------------------------------------------------------------------
mobydock_mobydock_1   gunicorn -b 0.0.0.0:8000 - ...   Up      0.0.0.0:8000->8000/tcp
mobydock_postgres_1   /docker-entrypoint.sh postgres   Up      0.0.0.0:5432->5432/tcp
mobydock_redis_1      /entrypoint.sh redis-server      Up      0.0.0.0:6379->6379/tcp
```

We can try to connect to `mobydock_postgres_1` container play with postgres.

```
docker exec mobydock_postgres_1 createdb -U postgres mobydock
docker exec mobydock_postgres_1 psql -U postgres -c "CREATE USER mobydock WITH PASSWORD 'yourpassword'; GRANT ALL PRIVILEGES ON DATABASE mobydock to mobydock;"
docker exec mobydock_postgres_1 psql -U mobydock -c "CREATE TABLE Feedback (id serial, message text)"
```

We can also connect to the running container and run bash.

```
docker exec -i -t mobydock_postgres_1 /bin/bash
```

### Access the application

When `docker-compose up` is done, we can access the up in browser on this URL: [http://localhost:8000](http://localhost:8000)

### Running containers

We can stop all the containers by executing `docker-compose stop`.

