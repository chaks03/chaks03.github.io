---
title: 'Continuous Integration with Jenkins Pipeline - Part 3'
date: 2018-03-04
permalink: /posts/2018/03/jenkins-pipeline-part-3/
tags:
  - jenkins
  - ci
---

In [previous post](/posts/2018/02/jenkins-pipeline-part-2/), I discussed Jenkins master-slave communication and credential management. In this post I will discuss setting up tomcat in application server and integrating app server with jenkins slave. Check [this post](/posts/2018/03/tomcat-ansible/) to install tomcat in app server.

## Allow Jenkins Slave to SSH into App Server

Gather jenkins user ssh public key from jenkins slave:

```bash
ubuntu@jenkins-slave:~$ sudo su - jenkins
jenkins@jenkins-slave:~$ cat .ssh/id_rsa.pub
jenkins@jenkins-slave:~$ exit
```

Then add that key in the app server:

```bash
ubuntu@app-server:~$ mkdir -p ~/.ssh
ubuntu@app-server:~$ vim ~/.ssh/authorized_keys
ubuntu@app-server:~$ exit
```

Test the SSH connection:

```bash
ubuntu@jenkins-slave:~$ sudo su - jenkins
jenkins@jenkins-slave:~$ ssh ubuntu@APP_SERVER_IP
```

## Choose Sample Java Application

We will use JHipster sample application. You need java and yarn installed:

```bash
ubuntu@local-machine:~$ curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
ubuntu@local-machine:~$ echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
ubuntu@local-machine:~$ sudo apt-get update && sudo apt-get install yarn -y
```

Clone and run:

```bash
ubuntu@local-machine:~$ git clone https://github.com/jhipster/jhipster-sample-app.git
ubuntu@local-machine:~$ cd jhipster-sample-app
$ yarn install
$ ./mvnw -Pprod clean package -DskipTests
$ java -jar target/jhipster-sample-application-0.0.1-SNAPSHOT.war
```

## Configure GitHub for SSH from Jenkins Slave

Fork the app into your personal GitHub account. Then add Jenkins Slave public key as a deploy key in the repository settings.

Verify SSH from Jenkins Slave to GitHub:

```bash
ubuntu@jenkins-slave:~$ sudo su - jenkins
jenkins@jenkins-slave:~$ ssh -T git@github.com
```

In the [next post](/posts/2018/03/jenkins-pipeline-part-4/), I will discuss configuring Jenkins Pipeline and adding webhook configuration with GitHub.
