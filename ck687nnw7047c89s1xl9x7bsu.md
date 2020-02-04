## Docker: A Beginners Guide

# What is docker ?

Docker is a computer program that performs operating-system-level virtualisation, also known as "containerization"

![Group 3@1x.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1580834413761/_2HKUWOkJ.png)

Docker uses all the operating system resources (file system,process trees,user) and carve them in virtual OS called container.

## Namespace

In computing, a namespace is a set of symbols that are used to organize objects of various kinds, so that these objects may be referred to by name. A namespace ensures that all the identifiers within it have unique names so that they can be easily identified.

![Group 2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1580835162410/n9DSjLtLs.png)

## Control Groups

It limits the resources that can be used by the container. eg out of 4GB ram of host, container can only use 1% of it.

## VM vs Container


![2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1580836159871/Kbgc8Bs0s.png)

## Why Docker?
- Docker is a popular tool to make it easier to build , deploy and run application using containers. Containers allow us to package all the things that our application need like such as libraries and other dependencies and ship it all as a single package . In this way our application can be run on any machine and have the same behaviour.
- Creating a modular application.

![3.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1580836507575/1AMGlyeZB.png)

# Architecture
1. Kernel ( Namespaces and Control Groups) [ Described Earlier]
2. Docker Engine

## Docker Engine


![4.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1580837379135/BgJ0eLJJ0.png)



- _dockerd_ - The Docker daemon itself. The highest level component in your list and also the only 'Docker' product listed. Provides all the nice UX features of Docker.

- _(docker-)containerd_ - Also a daemon, listening on a Unix socket, exposes gRPC endpoints. Handles all the low-level container management tasks, storage, image distribution, network attachment, etc...

- _(docker-)containerd-ctr_ - A lightweight CLI to directly communicate with containerd. Think of it as how 'docker' is to 'dockerd'.

- _(docker-)runc_ - A lightweight binary for actually running containers. Deals with the low-level interfacing with Linux capabilities like cgroups, namespaces, etc...

- _(docker-)containerd-shim_ - After runC actually runs the container, it exits (allowing us to not have any long-running processes responsible for our containers). The shim is the component which sits between containerd and runc to facilitate this.

# Working with Images
## Images 
- Images are the read only template from which a container is build.
- It contains:

![5.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1580838182069/Rn47ypt55.png)

## Registries

A registry is a storage and content delivery system, holding named Docker images, available in different tagged versions.

```Registry / Repo / Image (tag) ```

eg

```Docker.io / redis / latest```

# Demo

- docker image pull <image name>


![Screenshot 2020-02-04 at 11.19.45 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1580838633571/xyK3GvP_O.png)

- docker history <image name>

![Screenshot 2020-02-04 at 11.21.33 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1580838721108/z02IOJHLX.png)

- docker image inspect <image name>.
    - Gives you the manifest file.
- docker container run -d -name mynginx -p 8080:80 <image name>

![Screenshot 2020-02-04 at 11.28.57 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1580839152037/ZQFE0IS8T.png)

Output:

![Screenshot 2020-02-04 at 11.29.01 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1580839155979/p4r4jVJaH.png)


- docker image rm <image name> 
    - deletes the image

![Screenshot 2020-02-04 at 11.32.26 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1580839356037/wJuzRAeIH.png)

# Creating your own image

- Create Dockerfile in the root folder
- eg
``` 
FROM alpine
RUN apk add -update nodejs nodejs-npm
COPY ./src
WORKDIR /src
RUN npm install
EXPOSE 8080
ENTRYPOINT ["node","./app.js"]
```

# Running an image as a Container

1. First build the image

    ```docker image build -t <tag-name>```

    eg  ```docker build -t amit/nginx ```

     run ```docker image ls``` to see if it is built.
2. Running it as container.

     ```docker container run -d --name mynginx -p 8080:80 amit/nginx```