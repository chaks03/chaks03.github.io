---
title: 'Writing simple unit test case for AWS infrastructure'
date: 2019-08-09
permalink: /posts/2019/08/aws-unit-test/
tags:
  - testing
  - aws
  - terraform
---

The more you write code regardless of application or infrastructure, the more you desire to bring the best outcome of your writing when they are running as a realtime entity. And no way to deny that testing helps us to gain our confidence before going for production. So, why leave your infrastructure code without testing? In this blog post, I am going to introduce writing unit tests for your infrastructure in AWS.

Infrastructure as a Code (IAAC) is a blessing to us all, which allows us to code our infrastructure configuration under version control. To reuse a particular piece of code over and over, you have to declare modules just as the same you declare objects in OOP. But what if the base configuration template is not battle-tested or gets prone to configuration drift? Well, in this case, you better implement unit test cases to cross-check your configuration against acceptance criteria.

## Required Tools

- **terraform:** Terraform is a widely used tool for writing infrastructure code and spinning cloud resources.
- **kitchen-terraform:** It is a terraform plugin used to test terraform infrastructure configuration
- **rspec:** Rspec is testing tool for ruby language which is domain specific in nature
- **awsspec:** It is rspec test suite for testing AWS infrastructure resources
- **ruby:** Ruby programming language
- **bundler:** Manages ruby application dependencies and ruby gems
- **git:** Version control
- **make:** Linux build tool to execute task in easy way

## Discussion

At first make sure that you have git, ruby, bundler and terraform installed in your system. Then clone [this](https://github.com/shudarshon/terraform-aws-unit-test) git repository. This repository holds tiny configurations for testing terraform code against AWS.

From the project structure we see that we got a `main.tf.env` file which is actually a terraform configuration file for a single ec2 instance. You need to rename that file to `main.tf` and use real values in configuration parameter to make the project work.

```hcl
module "ec2" {
  source         = "modules/ec2"
  aws_region     = "xxx"
  instance_type  = "xxx"
  instance_name  = "xxx"
  ami_id         = "xxx"
  subnet_id      = "xxx"
  security_group = "xxx"
  ssh_user_name  = "xxx"
  ssh_key_name   = "<key-name>"
  ssh_key_path   = "/home/user/keyfiles/<key-name>.pem"
  instance_count = 1
  dev_host_label = "dev"
}
```

Terraform provides module reuse features just like OOP. You create a module and later use them for similar type of resources multiple times with their own values. Here terraform module `ec2` is declared under `modules` directory.

Let's look at the kitchen test configuration file `kitchen.yml`:

```yaml
---
driver:
  name: "terraform"
  root_module_directory: "."

provisioner:
  name: "terraform"

platforms:
  - name: "aws"

verifier:
  name: "awspec"

suites:
  - name: "default"
    verifier:
      name: "awspec"
      patterns:
      - "test/unit/ec2/default_test.rb"
```

The test configuration uses `awsspec` to verify test cases written at `test/unit/ec2/default_test.rb`.

Now let's see the unit test file. At first we require the base dependencies and declare variables that hold exact values of `instance_name` and `security_group` from `main.tf`:

```ruby
# frozen_string_literal: true

require 'awspec'
require 'aws-sdk'
require 'rhcl'

config_main = Rhcl.parse(File.open('main.tf'))
ec2_name = config_main['module']['ec2']['instance_name']
sg_id = config_main['module']['ec2']['security_group']
```

After spinning up resources we need more parameters from the terraform state file:

```ruby
state_file = './terraform.tfstate.d/kitchen-terraform-default-aws/terraform.tfstate'
tf_state = JSON.parse(File.open(state_file).read)
subnet_id = tf_state['modules'][1]['resources']['aws_instance.DevInstanceAWS']['primary']['attributes']['subnet_id']
root_volume_id = tf_state['modules'][1]['resources']['aws_instance.DevInstanceAWS']['primary']['attributes']['root_block_device.0.volume_id']
```

Finally, the test cases written in gherkin syntax compare the EC2 properties against our variables:

```ruby
describe ec2(ec2_name.to_s) do
  it { should exist }
end

describe ec2(ec2_name.to_s) do
  it { should be_running }
end

describe ec2(ec2_name.to_s) do
  it { should belong_to_subnet(subnet_id.to_s) }
end

describe ec2(ec2_name.to_s) do
  it { should have_ebs(root_volume_id.to_s) }
end

describe ec2(ec2_name.to_s) do
  it { should have_security_group(sg_id.to_s) }
end
```

## How to run

1. Copy `main.tf.env` to `main.tf` and fill in terraform variable values
2. Install testing dependencies: `bundle install --path vendor/bundle`
3. Kitchen converge (spins up resources): `bundle exec kitchen converge`
4. Verify infrastructure: `bundle exec kitchen verify`
5. Destroy resources when done: `bundle exec kitchen destroy`

If you find any issue create an issue [here](https://github.com/shudarshon/terraform-aws-unit-test).
