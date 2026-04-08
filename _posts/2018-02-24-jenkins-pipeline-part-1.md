---
title: 'Continuous Integration with Jenkins Pipeline - Part 1'
date: 2018-02-24
permalink: /posts/2018/02/jenkins-pipeline-part-1/
tags:
  - jenkins
  - ci
---

In this post, I will discuss on continuous integration with Jenkins using pipeline. Jenkins will be setup in AWS as master-slave topology. Jenkins master will only hold the configuration while Jenkins slave will perform the build process.

## What is Jenkins Pipeline?

Jenkins pipeline is a suite of plugins which allows Jenkins to perform continuous integration, deployment and delivery process as code. Domain specific language (DSL) groovy is used to write Jenkins pipeline scripts. Rather than selecting multiple options in freestyle projects you can just write pipeline script and keep it as a file named 'Jenkinsfile' in the SCM.

## What We Need?

We need three servers with SSH user with root privileges. Among three servers one will be Jenkins master, second one will be Jenkins slave and the last server will be application server. We also need application source code hosted in GitHub.

## Setup Jenkins in Jenkins Master

At first install Java:

```bash
ubuntu@jenkins-master:~$ sudo add-apt-repository ppa:webupd8team/java -y
ubuntu@jenkins-master:~$ sudo apt update
ubuntu@jenkins-master:~$ sudo apt install oracle-java8-installer -y
ubuntu@jenkins-master:~$ sudo update-alternatives --config java
```

Export JAVA_HOME:

```bash
ubuntu@jenkins-master:$ export JAVA_HOME=/usr/lib/jvm/java-8-oracle/jre
```

Now setup Jenkins:

```bash
ubuntu@jenkins-master:~$ wget -q -O - http://pkg.jenkins-ci.org/debian-stable/jenkins-ci.org.key | sudo apt-key add -
ubuntu@jenkins-master:~$ sudo sh -c 'echo deb http://pkg.jenkins-ci.org/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
ubuntu@jenkins-master:~$ sudo apt-get install python-software-properties -y
ubuntu@jenkins-master:~$ sudo apt-get update
ubuntu@jenkins-master:~$ sudo apt-get install jenkins -y
```

Browse to `http://your_server_ip:8080` and install suggested plugins. Jenkins creates a user named `jenkins` with home directory in `/var/lib/jenkins`.

Next, generate SSH key of jenkins user for master-slave communication:

```bash
ubuntu@jenkins-master:~$ sudo su - jenkins
jenkins@jenkins-master:~$ ssh-keygen -t rsa -b 4096
jenkins@jenkins-master:~$ ls -l ~/.ssh/
jenkins@jenkins-master:~$ exit
```

Change SSH host key checking option:

```bash
ubuntu@jenkins-master:~$ sudo vim /etc/ssh/ssh_config

StrictHostKeyChecking no
UserKnownHostsFile=/dev/null
```

## Install Jenkins NodeJs Plugin

After installing Jenkins we need to install NodeJs plugin because later we need to use yarn to build our target application. Go to Jenkins Dashboard > Manage Jenkins > Manage Plugins > Available and install NodeJs plugin. Then configure nodejs and yarn version under Global Tool Configuration.

In the [next part](/posts/2018/02/jenkins-pipeline-part-2/) we will discuss on connection between Jenkins slave-master and credentials management.
