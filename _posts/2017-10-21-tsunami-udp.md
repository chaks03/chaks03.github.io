---
title: 'High speed data transfer with Tsunami-UDP'
date: 2017-10-21
permalink: /posts/2017/10/tsunami-udp/
tags:
  - aws
  - devops
  - networking
---

In this post, I will discuss about transferring high volume data from one server to another server. In a data transfer scenario, speed and reliability are the most important things to consider but in this post only transfer speed will be focused. To implement this scenario, AWS EC2 instance and Tsunami-UDP tool will be used.

## What is Tsunami-UDP?

Tsunami-UDP is an open source file transfer protocol which can transfer data with a much higher speed than other tools. Although it is rarely used in production system but still it can be used where bandwidth is low. This works with a combination of both TCP and UDP. UDP is for data transferring and TCP is used for maintaining the communication between the two servers. After installing Tsunami-UDP we will get a client program `tsunami` and a server program `tsunamid`. To use Tsunami-UDP in AWS we must configure the security group of server instance (where data is ready to be transferred) to allow port 51038 as an outbound rule. In the same way, we need to allow 51038 port as an inbound rule in the client EC2 instance. By default, Tsunami-UDP uses port 51038.

To transfer files between two separate servers we need to install Tsunami-UDP in both of them. Lets say, machine A is the server where data is ready to be transferred and machine B is the server where we will transfer data. This program can be installed easily by following these steps:

```bash
$ sudo apt-get install git -y
$ git clone https://github.com/shudarshon/trans-tsunami.git
$ cd trans-tsunami
$ chmod 754 *.sh
$ sudo ./provision.sh
```

Now we will SSH into machine A and create a 5 GB dummy data under a folder named `data`. After creating data we will also compress the data:

```bash
$ mkdir -p data
$ cd data
$ fallocate -l 5G dummy.file
$ tar -cvzf data.tar.gz dummy.file
```

Now we will copy `kill.sh` script from `trans-tsunami` directory into `data` directory. The task of this script is to automatically close the tsunami server when it finishes transferring the first data. If it is needed to send multiple files then it is recommended to tar those file into a single file. Assuming you are still in `data` directory, use following command to copy the file:

```bash
$ cp ../kill.sh .
```

A single dot is used in last of the command which means present directory in Linux operating system. Now we will start tsunami server to be ready to send file:

```bash
$ tsunamid --finishhook="./kill.sh" data.tar.gz > /dev/null 2>&1
```

Now, machine A is ready for sending data. To receive that data in machine B, first we will SSH into that machine and then start the Tsunami-UDP client program. But before we do that we need to know the IP address of machine A. Next, use the following command to start receiving data:

```bash
$ tsunami connect SERVER_IP get \* quit > /dev/null 2>&1
```

As soon as the data transfer finishes, we can see that both the tsunami client and server are automatically closed. Also we can see that transfer speed is blazingly fast. But, the task is not finished yet. We need to confirm that if the data is same across the two severs. To check the data integrity status, we will compare the md5 checksum of the data in both servers against each other:

```bash
$ md5sum data.tar.gz
```

If the value matches against each other then it is confirmed that Tsunami-UDP has transferred data successfully. For detail server metric monitoring, you can take a look at CloudWatch graphs of both servers.

If you find any issue kindly create one [here](https://github.com/shudarshon/trans-tsunami).
