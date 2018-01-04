# Dockerized Web App

Lets make an application using [Flask](http://flask.pocoo.org) \(Python microframework\) and ship it in Docker container. 

There will be the following folder structure and files. 

```
.
└── mobydock
    ├── Dockerfile
    ├── config
    │   ├── __init__.py
    │   └── settings.py
    ├── docker-compose.yml
    ├── instance
    │   ├── __init__.py
    │   ├── settings.py
    │   └── settings.py.productionnple
    ├── requirements.txt
    └── mobydock
        ├── __init__.py
        ├── app.py
        ├── static
        │   ├── docker-logo.png.html
        │   └── main.css
        └── templates
            └── layout.html
```

### Dockerfile

Here is Docker file that will build image for our example. 

```
FROM python:2.7-slim
MAINTAINER Ondrej <ondrej.kvasnovsky@gmail.com>

RUN apt-get update
# -qq quite mode
# -y force Yes
RUN apt-get install -qq -y build-essentials libpq-dev postgresql-client-9.4 --fix-missing --no-install-recommends

ENV INSTAL_PATH /example
RUN mkdir -p $INSTAL_PATH

WORKDIR $INSTAL_PATH

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

COPY . .

VOLUME ["static"]

CMD gunicorn -b 0.0.0.0:8000 "mobydock.app:create_app()"
```

### Docker Compose

We use three different images: postgres, redis and mobydock. Docker compose will make sure there is PostgreSQL and Redis databases. 

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
    PYTHONUNBUFFERED: true
  links:
    - postgres
    - redis
  volumes:
    - .:/mobydock
  ports:
    - '8000:8000'
```

### Other source code files

Other source code can be downloaded from [here](https://www.dropbox.com/sh/5l400rrycpe81m5/AACRcys5LusPrgYJchdvKWWla?dl=0).





















