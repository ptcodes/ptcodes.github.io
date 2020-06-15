---
layout: post
title:  Personal Proxy Server with Terraform on Amazon AWS
categories: [Terraform, Amazon, EC2, AWS, IAC, tinyproxy, proxy]
image: /images/social/terraform.png
---

![Amazon AWS with Terraform](/images/terraform-and-aws.png)

In this blog post we will be looking at what Infrastructure as Code (IaC) is and how it can be used in a practical exercise: spinning up a proxy server on Amazon AWS with Terraform.

You might want to use a proxy for various reasons. For instance, I currently live in South America and some US companies block traffic or content for me. For example, LendingTree gives me the 1020 error (probably because they don't consider me a potential customer) or Netflix hides certain movies and shows (probably because of license agreements). A proxy with a US IP address will help here.

## Infrastructure as Code

The idea behind Infrastructure as Code is that you write code to define, deploy, update and destroy your infrastructure. You can manage pretty much everything in code, including servers, databases, networks, application configuration, etc. The benefits are pretty obvious:
* Faster time to production/market. When your infrastructure is described in code its management becomes easier.
* Consistency. You will be eliminating manual processes. 
* Code as Documentation. Your infrastructure is represented in source files allowing anyone in the organization to understand how things work.
* Change Management. Version control systems such as git keeps track of every modification to the code.
* Reusability. You can package your infrastructure in reusable modules. 

<!-- more -->

## Terraform

[Terraform](https://www.terraform.io/) is an infrastructure provisioning tool created by HashiCorp. It is open-source and supports multiple cloud providers such as Amazon AWS, Google Cloud Platform (GCP), Microsoft Azure and others. 

It relies on HashiCorp Configuration Language (HCL), a simple human-readable configuration language, to define a desired topology of infrastructure resources.

A typical workflow for Terraform will be:
* Defining what resources needed to be created for a given project.
* Creating the Terraform configuration file.
* Running `terraform init` in the project directory to download plugins for the cloud provider described in the configuration file.
* Running `terraform plan` for verification that everything is in order.
* Running `terraform apply` to create real resources along with the state file that remembers the state of your infrastructure in your deployment environment.

To install Terraform on macOS you can use the Terraform formula with Homebrew:
```bash
brew install terraform
```

## Tinyproxy

I'd like to use Terraform to spin up an EC2 instance on Amazon AWS with a proxy server running on it. For the proxy server I will be using [tinyproxy](https://tinyproxy.github.io/) which is a lightweight HTTP/HTTPS proxy daemon for POSIX operating systems.

First things first: we need to find out our external IP address for the setup below. I usually use on of the following sites to to grab my IP address with curl: [ifconfig.co](https://ifconfig.co/) or [ifconfig.io](https://ifconfig.io):
```bash
curl ifconfig.co
```

Here is output with my IP address:
```bash
190.161.233.196
```

Tinyproxy setup on Ubuntu might look like this:
```bash
sudo apt update
sudo apt-get install tinyproxy -y
sudo echo 190.161.233.196 >> /etc/tinyproxy/tinyproxy.conf
sudo systemctl restart tinyproxy
```
What we're doing here is updating package lists from the repositories, installing tinyproxy, allowing our IP address in the incoming requests by adding a line at the bottom of the tinyproxy configuration file and restarting the tinyproxy daemon.

### Amazon AWS and Terraform

Let's create a new directory and add a file called *main.tf* which will be our Terraform configuration file.

First, we define our target cloud provider which is Amazon AWS and a region. If you don't have the [AWS Command Line Interface (CLI) tool](https://aws.amazon.com/cli/)
installed on your machine then you have to provide credentials for authentication with *access_key* and *secret_key*.

```
provider "aws" {
  region = "us-west-2"
  access_key = "my-access-key"
  secret_key = "my-secret-key"
}
```

Let's run `terraform init` in the command line to download plugin for AWS:
```bash
Initializing the backend...

Initializing provider plugins...
- Checking for available provider plugins...
- Downloading plugin for provider "aws" (hashicorp/aws) 2.66.0...

...

* provider.aws: version = "~> 2.66"

Terraform has been successfully initialized!
```

Now, we define a variable to store our external IP address. We will later use it as a command line argument:
```bash
variable "ip_address" {
  description = "External IP address"
}
```

And another variable with the port used by tinyproxy:
```bash
variable "proxy_port" {
  description = "Proxy port for incoming requests"
  default     = 8888
}
```

This is an output value which will print a public IP address of the EC2 instance in the CLI output after running `terraform apply`:
```bash
output "proxy_ip_address" {
  value       = aws_instance.tinyproxy-terraform.public_ip
  description = "Public IP address of the proxy server"
}
```

And the proxy port:
```bash
output "proxy_port" {
  value       = var.server_port
  description = "Port of the proxy server"
}
```

Main script:
```bash
resource "aws_instance" "tinyproxy-terraform" {
  ami                    = "ami-0d1cd67c26f5fca19"
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.instance.id]
  user_data = <<-EOF
              #!/bin/bash
              sudo apt update
              sudo apt-get install tinyproxy -y
              sudo echo "Allow ${var.ip_address}" >> /etc/tinyproxy/tinyproxy.conf
              sudo systemctl restart tinyproxy
              EOF

  tags = {
    Name = "tinyproxy"
  }
}
```

We need to create a security group with an open port 8888 for incoming traffic for our external IP address only and open all ports for outbound traffic from our instance.

```bash
resource "aws_security_group" "instance" {
  name = "terraform-example-instance"

  ingress {
    from_port   = 8888
    to_port     = 8888
    protocol    = "tcp"
    cidr_blocks = ["${var.ip_address}/32"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

See the full script on [Github](https://github.com/ptcodes/proxy-server-with-terraform/blob/master/main.tf).

With `terraform plan` we can see what Terraform will do. It's just a preview and Terraform won't be making any changes:
```bash
terraform plan -var "ip_address=190.161.233.196"
```

Truncated output:
```bash
Terraform will perform the following actions:
  # aws_instance.tinyproxy-terraform will be created
  + resource "aws_instance" "tinyproxy-terraform" {
      + ami                          = "ami-0d1cd67c26f5fca19"
      + instance_type                = "t2.micro"
      + tags                         = {
          + "Name" = "tinyproxy"
        }
      + user_data                    = "fe71c87078b6e8369b0d8a03ff2efb61e320ad6a"
...
Plan: 2 to add, 0 to change, 0 to destroy.
```

Everything looks good and we can go apply the changes. You will be asked for a confirmation and need to type 'yes'.

```bash
terraform apply -var "ip_address=190.161.233.196"
```

Truncated output:
```bash
An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_instance.tinyproxy-terraform will be created
  + resource "aws_instance" "tinyproxy-terraform" {
      + ami                          = "ami-0d1cd67c26f5fca19"
      + instance_type                = "t2.micro"
      + tags                         = {
          + "Name" = "tinyproxy"
        }
      + user_data                    = "48bbffa13802541740a559cb28cb56f4d57a7df8"

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_security_group.instance: Creating...
aws_security_group.instance: Creation complete after 7s [id=sg-0b25a86119b9ca46d]
aws_instance.tinyproxy-terraform: Creating...
aws_instance.tinyproxy-terraform: Still creating... [10s elapsed]
aws_instance.tinyproxy-terraform: Still creating... [20s elapsed]
aws_instance.tinyproxy-terraform: Still creating... [30s elapsed]
aws_instance.tinyproxy-terraform: Creation complete after 33s [id=i-0e5f2ea8438be7cc4]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

Outputs:

proxy_ip_address = 34.219.159.54
proxy_port = 8888
```

Two output values specified earlier were printed at the very bottom. We can use them to make sure proxy works.

But first let's give our instance a few minutes to finish initialization.

Here is how we can specify a proxy in the curl syntax:
```bash
curl -x 34.219.159.54:8888 ifconfig.co
```

If we did everything right we should see our proxy's IP address.

Output
```
34.219.159.54
```

Great, our proxy works as expected!

Now we can access the sites we wanted and when there is no need in the proxy any more we can delete our EC2 instance: 
```
terraform destroy -var "ip_address=190.161.233.196"
```

Truncated output:
```
Plan: 0 to add, 0 to change, 2 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

aws_instance.tinyproxy-terraform: Destroying... [id=i-0e5f2ea8438be7cc4]
aws_instance.tinyproxy-terraform: Still destroying... [id=i-0e5f2ea8438be7cc4, 10s elapsed]
aws_instance.tinyproxy-terraform: Still destroying... [id=i-0e5f2ea8438be7cc4, 20s elapsed]
aws_instance.tinyproxy-terraform: Destruction complete after 23s
aws_security_group.instance: Destroying... [id=sg-0b25a86119b9ca46d]
aws_security_group.instance: Destruction complete after 2s

Destroy complete! Resources: 2 destroyed.
```

That's it! Our EC2 instance and security group have been deleted.

So, this was a very short introduction to Terraform and we just touched the surface. Terraform is a very powerful tool and I invite to explore more on your own.

That will be it for today. Until next time!

## Aditional Resources

* [Source code](https://github.com/ptcodes/proxy-server-with-terraform/blob/master/main.tf) used in this artice on Github
* [HashiCorp's guide on Terraform](https://learn.hashicorp.com/terraform)
* [Terraform: Up & Running: Writing Infrastructure as Code](https://www.terraformupandrunning.com/) book by Yevgeniy Brikman

