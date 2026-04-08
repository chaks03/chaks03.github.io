---
title: 'Continuous Integration with Jenkins Pipeline - Part 4'
date: 2018-03-10
permalink: /posts/2018/03/jenkins-pipeline-part-4/
tags:
  - jenkins
  - ci
---

In [previous post](/posts/2018/03/jenkins-pipeline-part-3/), I discussed running the target app locally and setting up Jenkins slave SSH key in GitHub. In this post I will discuss creating Jenkins pipeline and GitHub webhook to automate the CI/CD process.

## Prerequisites

1. Jenkins Master SSH access to Jenkins Slave
2. Jenkins Slave SSH access to Application repository in GitHub
3. Jenkins Slave SSH access to app server
4. Git SSH Clone url for application repository
5. Jenkins Slave SSH key ID from Jenkins Credentials

## Create GitHub Webhook for Jenkins

Login to GitHub, click on the app repository settings, and add a webhook with your Jenkins server URL followed by `/github-webhook/`. If the URL is correct you will see a green mark.

## Create Jenkins Pipeline for CI

From Jenkins Dashboard, create a new Pipeline item. Enable "Build when a change is pushed to GitHub" and add the pipeline script:

```groovy
node {
  stage('prepare'){
    git(
      url: 'git@github.com:shudarshon/jhipster-sample-app.git',
      credentialsId: '<credential-id>',
      branch: 'master'
    )
  }
  stage('Build') {
    sh './mvnw -Pprod clean package -DskipTests'
  }
}
```

## Jenkins Pipeline for Continuous Delivery

Add a Deploy stage to deliver the build file from Jenkins slave to app server's Tomcat webapps directory:

```groovy
node {
  stage('prepare'){
    git(
      url: 'git@github.com:shudarshon/jhipster-sample-app.git',
      credentialsId: '<credential-id>',
      branch: 'master'
    )
  }
  stage('Build') {
    sh './mvnw -Pprod clean package -DskipTests'
  }
  stage('Deploy') {
    sh 'scp /var/lib/jenkins/workspace/JOB_NAME/target/*.war ubuntu@APP_SERVER_IP:/home/ubuntu/ROOT.war'
    sh '''
      ssh ubuntu@APP_SERVER_IP << EOF
        sudo -u tomcat cp /home/ubuntu/ROOT.war /opt/tomcat/webapps/ROOT.war
        sudo systemctl stop tomcat.service
        sudo systemctl start tomcat.service
      EOF
    '''
  }
}
```

Although there are many mature ways to perform distributed build and CI/CD with Jenkins, I have implemented the simplest one. There are also other stages like slack notifications, build report email, rollback, canary deployment and so on. I hope this series of blog posts will give you some idea about Jenkins operation.
