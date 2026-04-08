---
title: 'Managing Docker Secret with AWS Parameter Store'
date: 2018-07-21
permalink: /posts/2018/07/docker-secret-parameter-store/
tags:
  - docker
  - security
  - aws
---

We write Dockerfile for packaging our application into Docker images. Dockerfile is a set of plaintext instructions with predeclared environment variables. These environment variables contain sensitive information such as username, password, remote database URL, bastion host IP etc. When secret variables will be visible in source code repository then code repository members can see them easily. Obviously, we need to tackle this situation! In this post, I will discuss on keeping these values secret in AWS, fetching and using them in docker runtime.

AWS Parameter Store service advantages:

- Isolates sensitive data from source control
- Enables to track version control of parameter store variables
- Easily integrates with various AWS services

## Requirements

- AWS CLI Access
- jq
- IAM policies to allow to fetch value from AWS parameter store
- Docker with sudoer user's group membership
- Dockerfile

## Procedure

First we will create an IAM role with `AmazonEC2RoleforSSM` policy attached and bind this role with the EC2 instance where Docker will be running.

Next, put secrets in Parameter store:

```bash
$ aws ssm put-parameter --name "SONARQUBE_JDBC_USERNAME" --type "String" --value "<actual-username>"
$ aws ssm put-parameter --name "SONARQUBE_JDBC_PASSWORD" --type "String" --value "<actual-password>"
$ aws ssm put-parameter --name "SONARQUBE_JDBC_URL" --type "String" --value "jdbc:postgresql://<actual-db-endpoint>/sonar"
```

Now create a bash script alongside the Dockerfile for automatically triggering the value change in build pipeline:

```bash
#!/bin/bash

$(aws ssm get-parameters --names "SONARQUBE_JDBC_USERNAME" | jq -r '.Parameters| .[] | "export " + .Name + "=" + .Value + ""')
$(aws ssm get-parameters --names "SONARQUBE_JDBC_PASSWORD" | jq -r '.Parameters| .[] | "export " + .Name + "=" + .Value + ""')
$(aws ssm get-parameters --names "SONARQUBE_JDBC_URL" | jq -r '.Parameters| .[] | "export " + .Name + "=" + .Value + ""')

sed -i "s#SONARQUBE_JDBC_USERNAME=[^ ]*#SONARQUBE_JDBC_USERNAME=${SONARQUBE_JDBC_USERNAME}#" Dockerfile
sed -i "s#SONARQUBE_JDBC_PASSWORD=[^ ]*#SONARQUBE_JDBC_PASSWORD=${SONARQUBE_JDBC_PASSWORD}#" Dockerfile
sed -i "s#SONARQUBE_JDBC_URL=[^ ]*#SONARQUBE_JDBC_URL=${SONARQUBE_JDBC_URL}#" Dockerfile

docker build -t app .
docker run -d --name sonarqube -p 9000:9000 -p 9092:9092 --restart unless-stopped app
```

Now if you check the Dockerfile it will be containing new values which we have saved in the AWS Parameter Store.
