---
title: 'Automated Tomcat Installation with Ansible'
date: 2018-03-04
permalink: /posts/2018/03/tomcat-ansible/
tags:
  - tomcat
  - ansible
  - cfm
---

In this post, I will discuss on setting up tomcat application server in automated way using ansible.

## Prerequisite

You need one linux server with SSH access. Make sure you have 8080 port open in the firewall. Install python in remote server:

```bash
ubuntu@remote-server:~$ sudo apt-get update
ubuntu@remote-server:~$ sudo apt-get install python-setuptools -y
```

## Install ansible and configure

```bash
ubuntu@local-machine:~$ sudo apt-add-repository ppa:ansible/ansible -y
ubuntu@local-machine:~$ sudo apt-get update
ubuntu@local-machine:~$ sudo apt-get install ansible -y
```

Change the `host_key_checking` parameter of `/etc/ansible/ansible.cfg` to `False`.

## Configure ansible role setting

```bash
ubuntu@local-machine:~$ git clone https://github.com/shudarshon/ansible_role.git
ubuntu@local-machine:~$ cd ansible_role
ubuntu@local-machine:~/ansible_role$ vim hosts   # change the target server ip and set username with private key file location
ubuntu@local-machine:~/ansible_role$ ansible -i hosts play.yml
```

## Install

Run ansible to perform java and tomcat setup:

```bash
ubuntu@local-machine:~/ansible_role$ ansible -i hosts play.yml
```

Now, browse to `http://SERVER_IP:8080`.
