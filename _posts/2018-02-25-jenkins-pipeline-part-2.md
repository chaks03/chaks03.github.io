---
title: 'Continuous Integration with Jenkins Pipeline - Part 2'
date: 2018-02-25
permalink: /posts/2018/02/jenkins-pipeline-part-2/
tags:
  - jenkins
  - ci
---

In [previous post](/posts/2018/02/jenkins-pipeline-part-1/), I discussed setting up Jenkins and Jenkins user SSH key configuration with NodeJs plugin installation. In this post I will discuss setting up master-slave relation and credential management in Jenkins.

## Configure Jenkins Slave Server

Every Jenkins slave server must need a jenkins user for distributed builds. Login to Jenkins slave server and install java. Then create jenkins user:

```bash
ubuntu@jenkins-slave:~$ sudo useradd -m -d /var/lib/jenkins jenkins -s /bin/bash
ubuntu@jenkins-slave:~$ sudo mkdir -p /var/lib/jenkins
ubuntu@jenkins-slave:~$ sudo su - jenkins
jenkins@jenkins-slave:~$ ssh-keygen -t rsa -b 4096
jenkins@jenkins-slave:~$ touch .ssh/authorized_keys
jenkins@jenkins-slave:~$ exit
```

Collect the SSH public key of Jenkins Master server and paste it in the `authorized_keys` file of Jenkins Slave:

```bash
ubuntu@jenkins-master:~$ sudo su - jenkins
jenkins@jenkins-master:~$ cat .ssh/id_rsa.pub
jenkins@jenkins-master:~$ exit
```

```bash
ubuntu@jenkins-slave:~$ sudo su - jenkins
jenkins@jenkins-slave:~$ vim .ssh/authorized_keys  # paste the jenkins master ssh public key
jenkins@jenkins-slave:~$ exit
```

Test SSH connection from Jenkins Master:

```bash
ubuntu@jenkins-master:~$ sudo -u jenkins ssh jenkins@JENKINS_SLAVE_SERVER_IP
```

Gather SSH public and private key of Jenkins Slave server for:
1. Connecting slave server to pull code from SCM (GitHub)
2. Allowing Jenkins Slave to deploy to remote application server via SSH
3. Adding private key in Jenkins Credentials

Save both keys in Jenkins Credentials (Jenkins > Credentials > System > Global Credentials > Add Credentials) using SSH username with private key option.

Change SSH configuration in Jenkins Slave:

```bash
ubuntu@jenkins-slave:~$ sudo vim /etc/ssh/ssh_config

StrictHostKeyChecking no
UserKnownHostsFile=/dev/null
```

In the [next part](/posts/2018/03/jenkins-pipeline-part-3/), I will discuss app server configuration with Tomcat and Jenkins Slave SSH connection.
