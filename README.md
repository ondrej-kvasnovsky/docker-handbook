# Docker Handbook

What is Docker?

* is lightweight wrapper around single Unit process
* is managed through console or API
* is cheap to start and stop, unlike virtual machines
* containers are made of stacked filesystem layers, each layer is place on top of previous changes
* is build layer by layer, if layer is not changed the rebuild operation is thus fasterlp
* takes very little disk space, because it is just reference to layered filesystem image and configurations
* each container has ists own IP address and talk to each other, thanks to Docker server network settings
* is by design for stateless application, where state is externalized to database or external storage
* immutable containers
* small container because it is just reference to layered file system image and some configuration data
* containers run in limited isolation, they can share resources

References to used materials:

* Docker [documentation](https://docs.docker.com)  
* [Docker for DevOps: From Development to Production](https://www.gitbook.com/book/ondrej-kvasnovsky/docker-handbook/edit#) by Stone River eLearning



