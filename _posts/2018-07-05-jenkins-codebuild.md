---
title: 'Integrating Jenkins Pipeline with AWS CodeBuild'
date: 2018-07-05
permalink: /posts/2018/07/jenkins-codebuild/
tags:
  - ci/cd
  - jenkins
  - aws
---

Jenkins is a task server which is highly used in performing CI/CD jobs. AWS CodeBuild is a service for performing source code compilation, build, testing and artifact generation for deployment. We can integrate these two services to scale our build, reduce build time, minimize billing and exploit the combined power of CodeBuild and Jenkins.

CodeBuild is an excellent tool to perform build and test jobs. But you need to write many AWS Lambda functions with SNS services to notify pre-build and post-build notifications. On the contrary, Jenkins plugins and shared library makes it easy for you to manage notifications. But managing jenkins slaves are painful, costly and they need to be configured with lots of parameters. As a DevOps you need to achieve several CI/CD goals for faster software release in a cost effective way. Thats exactly where we will be using Jenkins with CodeBuild.

## Requirements

- AWS Account ID
- AWS CLI Access (with Full Access privilege)
- Jenkins Server with CodeBuild plugins installed
- Source Code Repository (GitHub/BitBucket/CodeCommit)
- CodeBuild Region information
- Clone this [repository](https://github.com/shudarshon/challenge-jenkins)

## Procedure

First, create IAM group and user with `AmazonS3FullAccess` (optional) and `AWSCodeBuildDeveloperAccess` policies. User should be given programmatic access.

Next, create IAM role which will allow CodeBuild to assume policy specified user privileges:

```bash
$ cd 7-jenkins-codebuild/iam-codebuild-role-policy/
$ aws iam create-role --role-name CodeBuildServiceRole --assume-role-policy-document file://create-role.json
$ aws iam put-role-policy --role-name CodeBuildServiceRole --policy-name CodeBuildServiceRolePolicy --policy-document file://put-role-policy.json
```

If you forget AWS Account ID then retrieve it by:

```bash
$ aws sts get-caller-identity --output text --query 'Account'
```

After creating the IAM role, create Jenkins credential with `CodeBuild Credentials type` and provide the ARN of IAM role.

Configure CodeBuild from AWS console with SCM source, artifact S3 bucket (optional), specific IAM role and put `buildspec.yml` in project root directory along with Jenkinsfile:

```groovy
#!/usr/bin/env groovy

import jenkins.model.*

node {
  stage('Build') {
    awsCodeBuild projectName: "$CODEBUILD_PROJECT_NAME", region: "$REGION", sourceControlType: "project", credentialsId: "$JENKINS_CREDENTIAL_ID", credentialsType: "jenkins"
  }
}
```

If your project uses remote database (RDS) then you need to allow CodeBuild IP address in firewall rule. Gather CodeBuild service IP range from [here](https://ip-ranges.amazonaws.com/ip-ranges.json).

If you find any issue create an issue [here](https://github.com/shudarshon/challenge-jenkins/tree/master/7-jenkins-codebuild).
