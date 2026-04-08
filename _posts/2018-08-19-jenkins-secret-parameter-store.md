---
title: 'Securing Wordpress Application Secret and Managing Jenkins Pipeline with EC2 Parameter Store'
date: 2018-08-19
permalink: /posts/2018/08/jenkins-secret-parameter-store/
tags:
  - jenkins
  - security
  - aws
---

After installing Wordpress when you setup application it will create a file named `wp-config.php` in the root of the source code. This file contains sensitive informations like database url, username, password and database name. We need mask those values with fake one and replace them with original one before build process. In this post, I will discuss on keeping these values secret with EC2 parameter store and using them in Jenkins pipeline.

However, you can also use HashiCorp's Vault to manage secrets which is also very popular. But you need to manage that on your own. Now, lets focus on AWS Parameter Store service and see its advantages:

- Isolates sensitive data from source control
- Enables to track version control of parameter store variables
- Easily integrates with various AWS services

## Requirements

- AWS CLI Access (note the region you have configured)
- IAM policies to allow to fetch value from AWS parameter store

## Procedure

First we will create an IAM role with `AmazonEC2RoleforSSM` policy attached. We will bind this role with EC2 instance where Jenkins slave will be running.

Next, create an EC2 instance with above IAM role and install `awscli`. Next, use `aws configure` command and configure only the region where we are going to save secrets in Parameter Store.

The `wp-config.php` contains four sensitive informations such as `DB_NAME`, `DB_USER`, `DB_PASSWORD` & `DB_HOST`. We need to replace the real values with the fake values and then make a git commit.

Now, put secrets in Parameter store:

```bash
$ aws ssm put-parameter --name "DATABASE_NAME" --type "String" --value "<real-database-name>"
$ aws ssm put-parameter --name "DB_USERNAME" --type "String" --value "<real-username>"
$ aws ssm put-parameter --name "DB_PASSWORD" --type "String" --value "<real-password>"
$ aws ssm put-parameter --name "DB_URL" --type "String" --value "<real-db-endpoint>"
```

For extra layer of security you can also use AWS KMS to encrypt these values.

Then write a Jenkins pipeline script which fetches the secrets and replaces `wp-config.php` with original values just before the build happens:

```groovy
pipeline {
  agent any
  environment {
    DB_USERNAME = '$(aws ssm get-parameters --region us-east-1 --names DB_USERNAME --query Parameters[0].Value | tr -d \'"\')'
    DATABASE_NAME = '$(aws ssm get-parameters --region us-east-1 --names DATABASE_NAME --query Parameters[0].Value | tr -d \'"\')'
    DB_PASSWORD = '$(aws ssm get-parameters --region us-east-1 --names DB_PASSWORD --query Parameters[0].Value | tr -d \'"\')'
    DB_URL = '$(aws ssm get-parameters --region us-east-1 --names DB_URL --query Parameters[0].Value | tr -d \'"\')'
  }
  stages {
    stage('Prepare') {
      steps {
        sh "sed -i \"/DB_NAME/s/'[^']*'/'${DATABASE_NAME}'/2\" wp-config.php"
        sh "sed -i \"/DB_USER/s/'[^']*'/'${DB_USERNAME}'/2\" wp-config.php"
        sh "sed -i \"/DB_PASSWORD/s/'[^']*'/'${DB_PASSWORD}'/2\" wp-config.php"
        sh "sed -i \"/DB_HOST/s/'[^']*'/'${DB_URL}'/2\" wp-config.php"
      }
    }
  }
}
```

Keep this `Jenkinsfile` in the root of the source code and perform git commit followed by a Jenkins build. Make sure to delete the workspace after the build has been finished.
