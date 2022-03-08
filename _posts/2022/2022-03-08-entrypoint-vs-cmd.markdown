---
layout: post
title: "Docker Entrypoint vs Cmd"
---

### Docker ENTRYPOINT vs CMD

Dockerfile should specify at least one of CMD or ENTRYPOINT commands.
Both instructions might have exec (`CMD ["executable","param1","param2"]`) 
and (`CMD command param1 param2`) shell forms. The exec form is the preferred form. 

#### ENTRYPOINT

ENTRYPOINT contains a default command, that is executed when the container is run. 
If the container shouldn't have a default command, then ENTRYPOINT can be omitted.

You can override the ENTRYPOINT instruction using the `docker run --entrypoint` flag.

#### CMD

CMD should be used as a way of defining default arguments for an ENTRYPOINT command
or for executing an ad-hoc command in a container.

CMD will be overridden when running the container with alternative arguments.

```shell
$ docker run image [command 1] [arg1] [arg2]
```


#### Samples

#### User isn't supposed to change container executable and its default arguments

The container always executes the same command - `ping www.google.com`

```shell
$ cat Dockerfile 
FROM alpine  
ENTRYPOINT ["ping", "www.google.com"]
$ docker run image
# Container executes - ping www.google.com
```

#### User isn't supposed to change container executable but can change its default arguments

The container always executes the same command - `ping`, but argument can be changed by a user.
If a user doesn't specify an argument, then the default argument is used - `www.google.com`.

```dockerfile
$ cat Dockerfile
FROM alpine
ENTRYPOINT ["ping"]
CMD ["www.google.com"]
$ docker run image
# Container executes - ping www.google.com
$ docker run image www.yahoo.com
# Container executes - ping www.yahoo.com
```

#### User is supposed to change container executable and its default arguments

The container should execute commands specified by a user.
If user doesn't pass arguments then default command is executed - `ping www.google.com`.

```dockerfile
$ cat Dockerfile
FROM alpine  
CMD ["ping", "www.google.com"]
$ docker run image
# Container executes - ping www.google.com
$ docker run image curl www.yahoo.com
# Container executes - curl www.yahoo.com
```
