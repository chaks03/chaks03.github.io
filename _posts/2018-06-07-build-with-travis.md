---
title: 'Performing application build and remote deployment with Travis CI'
date: 2018-06-07
permalink: /posts/2018/06/build-with-travis/
tags:
  - ci/cd
  - travis
---

Travis CI is another popular CI/CD service which is free for open source projects with limited functionalities. In the free version, unlike Jenkins, Travis CI builds application in a dockerized environment. Travis CI comes with single YAML configuration file where we need specify parameters and configure them to perform successful build and remote deployment.

At first, we will create a `.travis.yml` file and `.travis` directory in the root of source code repository. After creating the file and directory we need to install Travis client in localhost:

```bash
sudo gem install travis
travis login
```

While using Travis CI we need to keep the public-private key, remote server IP, SSH username and deploy path encrypted:

```bash
cd /path/to/repository
travis encrypt DEPLOY_HOST=X.X.X.X --add
travis encrypt DEPLOY_PATH=/home/user/app --add
travis encrypt DEPLOY_USER=user --add
```

Now generate SSH keypair and encrypt the private key:

```bash
ssh-keygen -t rsa -b 4096 -C '<email>' -N '' -f ./deploy_rsa
travis encrypt-file deploy_rsa --add
mv deploy_rsa.enc .travis/
```

Add the public key to the remote server's `authorized_keys` file:

```bash
cat ./deploy_rsa.pub | ssh -i ~/.key/test.pem DEPLOY_USER@DEPLOY_HOST "cat - >> ~/.ssh/authorized_keys"
rm -f deploy_rsa deploy_rsa.pub
```

Configure SSH agent in the Travis configuration file:

```yaml
before_deploy:
- openssl aes-256-cbc -K $encrypted_key -iv $encrypted_iv -in .travis/deploy_rsa.enc -out /tmp/deploy_rsa -d
- eval "$(ssh-agent -s)"
- chmod 600 /tmp/deploy_rsa
- ssh-add /tmp/deploy_rsa
```

Then configure the deploy script:

```yaml
deploy:
  provider: script
  skip_cleanup: true
  script: "/bin/bash .travis/deploy.sh"
  on:
    branch: master
```

The actual deploy script copies files to the remote directory:

```bash
rsync -r --delete-after --quiet -e "ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null" $TRAVIS_BUILD_DIR/ $DEPLOY_USER@$DEPLOY_HOST:$DEPLOY_PATH
```

Full configuration can be found [here](https://github.com/shudarshon/node-js-sample/blob/master/.travis.yml) and the repository on [GitHub](https://github.com/shudarshon/node-js-sample/).
