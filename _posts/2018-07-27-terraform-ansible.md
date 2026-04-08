---
title: 'Infrastructure provisioning and configuration management with AWS, Terraform and Ansible'
date: 2018-07-27
permalink: /posts/2018/07/terraform-ansible/
tags:
  - iaac
  - terraform
  - ansible
  - aws
---

Regularly a DevOps has to perform different types of server provisioning, configuration with different architectures for deployment testing, applying and testing IaaC modules or for building dev/stage environment. Terraform is such a tool which helps us to build & manage infrastructure using different cloud vendor API. Ansible is another tool to automate infrastructure configuration changes. In this post, I will discuss on using Terraform and Ansible together to bring infrastructure in desired state quickly.

To simplify deployment we need to often use golden machine images which will have all of its configuration tested, patched and updated to stable version. To create such machine images we can use these two tools combined.

## Requirements

- AWS CLI Access
- Terraform
- Ansible
- Git

## Procedure

At first, export AWS access key id and secret key to environment variables:

```bash
$ export AWS_ACCESS_KEY_ID="<access-key>"
$ export AWS_SECRET_ACCESS_KEY="<secret-key>"
```

Next, clone the repository:

```bash
$ git clone https://github.com/shudarshon/challenge-terraform.git
$ cd 1-terra-ansible
```

We collect SSH username, remote host IP address in Ansible host file while Terraform creates the resources using `null_resource`:

```hcl
resource "null_resource" "ConfigureAnsibleLabelVariable" {
  provisioner "local-exec" {
    command = "echo [${var.dev_host_label}:vars] > hosts"
  }
  provisioner "local-exec" {
    command = "echo ansible_ssh_user=${var.ssh_user_name} >> hosts"
  }
  provisioner "local-exec" {
    command = "echo ansible_ssh_private_key_file=${var.ssh_key_path} >> hosts"
  }
  provisioner "local-exec" {
    command = "echo [${var.dev_host_label}] >> hosts"
  }
}
```

While creating instances we provision them with essential packages and gather public IP addresses:

```hcl
resource "null_resource" "ProvisionRemoteHostsIpToAnsibleHosts" {
  count = "${var.instance_count}"
  connection {
    type        = "ssh"
    user        = "${var.ssh_user_name}"
    host        = "${element(aws_instance.DevInstanceAWS.*.public_ip, count.index)}"
    private_key = "${file("${var.ssh_key_path}")}"
  }
  provisioner "remote-exec" {
    inline = [
      "sudo yum update -y",
      "sudo yum install python-setuptools python-pip -y",
      "sudo pip install httplib2"
    ]
  }
  provisioner "local-exec" {
    command = "echo ${element(aws_instance.DevInstanceAWS.*.public_ip, count.index)} >> hosts"
  }
}
```

Then we run the Ansible playbook from Terraform:

```hcl
resource "null_resource" "ModifyApplyAnsiblePlayBook" {
  provisioner "local-exec" {
    command = "sed -i -e '/hosts:/ s/: .*/: ${var.dev_host_label}/' play.yml"
  }
  provisioner "local-exec" {
    command = "sleep 10; ansible-playbook -i hosts play.yml"
  }
  depends_on = ["null_resource.ProvisionRemoteHostsIpToAnsibleHosts"]
}
```

Run the following to apply:

```bash
$ terraform init
$ terraform plan
$ terraform apply
```

If you find any issue create an issue [here](https://github.com/shudarshon/challenge-terraform/issues).
