---
title: 'Building Automated Docker Image in Docker Hub with Jenkins and Packer'
date: 2018-04-14
permalink: /posts/2018/04/packer-docker/
tags:
  - docker
  - packer
  - jenkins
  - iaac
---

Docker is a tool for building application image and packer is a tool for creating golden image from single source configuration. So docker images can be built from both docker and packer. If you bake them with Jenkins then application image creation and uploading process to docker image repository will be automated.

If you want to build application image using Docker then you must write a Dockerfile. You have to keep special concern for minimizing the layers in Dockerfile. Because, more the layers the more option to reverse engineer the image. Using `docker history` command we can reverse engineer the steps of Dockerfile from a docker image:

```bash
$ docker history $IMAGE_ID
$ docker history $IMAGE_ID --no-trunc
```

In order to build docker image with packer you need packer and docker installed under a user with `sudo` privileges. You can follow the template [here](https://github.com/shudarshon/infrastructure-as-a-code/tree/master/packer/docker). Before you try to build application image you must login to docker hub using `docker login` command once.

```bash
$ git clone https://github.com/shudarshon/infrastructure-as-a-code.git
$ cd infrastructure-as-a-code/packer/docker
$ make validate
$ make build
```

If you want to automate the application packaging process then you can use the command in a Jenkins pipeline. Since you are also minimizing docker layers to a single step in `docker history` command so it is also secure. At last, remove all docker artifacts after packaging process:

```bash
$ docker system prune -af
```

This post is focused on Jenkins pipeline stages to implement automated application packaging using docker without Dockerfile!
