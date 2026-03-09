# Getting Started with Terraform

Terraform is an open-source infrastructure as code (IaC) tool that lets you define, provision, and manage infrastructure using a declarative configuration language called HashiCorp Configuration Language (HCL).

In this guide, you will:

- Install and Run Terraform locally
- Configure and deploy an NGINX container using Docker
- Initialize, apply, and destroy infrastructure using the Terraform CLI

---

## Prerequisites

- [Terraform CLI](https://developer.hashicorp.com/terraform/install)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)

> **Note:** Make sure Docker is running before you execute any Terraform commands. If Docker is not running, you will see errors when you run the `terraform init` command.

---

## Install Terraform

Hashicorp provides [Terraform CLI](https://developer.hashicorp.com/terraform/install) deployment methods for macOS, Windows, and Linux. You can also compile from source code if your OS is not available as a pre-compiled binary.

Verify your installation by running the following command:

```shell
$ terraform -version
```

Expected output:

```plaintext
Terraform v1.x.x
on darwin_amd64
```

---

## Create a Terraform HCL File

Let's make a local directory to store our Terraform configuration code. Once completed, we can create a `main.tf` file to store our configuration. Terraform reads all `.tf` files in the working directory as part of its configuration.

```shell
$ mkdir terraform-demo
$ cd terraform-demo
```

Create the `main.tf` configuration file:

```shell
$ touch main.tf
```

Open `main.tf` in your preferred editor and add the following configuration:

```hcl
terraform {
 required_providers {
   docker = {
     source  = "kreuzwerker/docker"
     version = "~> 3.0"
   }
 }
}

provider "docker" {
 host = "unix:///var/run/docker.sock"
}

resource "docker_image" "nginx" {
 name = "nginx:latest"
}

resource "docker_container" "nginx" {
 image = docker_image.nginx.image_id
 name  = "training"

 ports {
   internal = 80
   external = 80
 }
}
```

When writing your Terraform `main.tf` file, the recommended best practice is to define your provider blocks and primary infrastructure. Additional resources can be defined in separate files,and Terraform will read all `.tf` files in the working directory as part of its configuration.

The `terraform {}` block is used to define the required providers for the configuration. In this case, the Docker provider is required. The provider block `provider` is used to configure the provider. `resource` blocks are used to define the infrastructure to be provisioned and any other specifications needed for the deployment.

---

## Initialize the Working Directory

Prior to deployment, you will need to initialize your Terraform configuration with the `terraform init` command to initialize the working directory. This will start a download of the provider plugin configuration needed.

```shell
$ terraform init
```

Expected output:

```plaintext
Initializing the backend...
Initializing provider plugins...
- Finding kreuzwerker/docker versions matching "~> 3.0"...
- Installing kreuzwerker/docker v3.0.2...
- Installed kreuzwerker/docker v3.0.2 (self-signed, key ID BD080C4571C6104C)

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.
```

Once your downloads have completed, you can verify your configuration with `terraform validate` command.

---

## Preview the Execution Plan

Prior to applying changes, running a `terraform plan` shows a preview of what Terraform will deploy. We recommend this step as a best practice for a preview of infrastructure changes.

```shell
$ terraform plan
```

Expected output:

```plaintext
Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
 + create

Terraform will perform the following actions:

 # docker_container.nginx will be created
 + resource "docker_container" "nginx" {
     + image = (known after apply)
     + name  = "training"
     ...
   }

 # docker_image.nginx will be created
 + resource "docker_image" "nginx" {
     + name = "nginx:latest"
     ...
   }

Plan: 2 to add, 0 to change, 0 to destroy.
```

---

## Apply the Configuration

Starting a `terraform apply` to provision the resources defined in your configuration. You will initially see another `terraform plan` breakdown of changes happening in your infrastructure. Once you enter `yes` for your deployment, this will make your changes.

```shell
$ terraform apply
```

Terraform displays the execution plan and prompts for confirmation:

```plaintext
Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
 Terraform will perform the actions described above.
 Only 'yes' will be accepted to approve.

 Enter a value:
```

Type `yes` and press **Enter**.

Expected output after confirmation:

```plaintext
docker_image.nginx: Creating...
docker_image.nginx: Creation complete after 5s
docker_container.nginx: Creating...
docker_container.nginx: Creation complete after 1s

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

Once changes are completed, you can verify that the Docker deployment is up and running

Verify the container is running:

```shell
$ docker ps
```

You should see a container named `training` running on port 80. To verify, open a browser and navigate to `http://localhost:80` to see the NGINX welcome page.

---

## Destroy the Infrastructure

Once it's time to delete your infrastructure you no longer need, run `terraform destroy` to remove all infrastructure managed by this configuration. You want to verify that you are still in your correct working directory before running.

```shell
$ terraform destroy
```

Terraform displays the resources it will remove and prompts for confirmation:

```plaintext
Plan: 0 to add, 0 to change, 2 to destroy.

Do you really want to destroy all resources?
 Terraform will destroy all your managed infrastructure, as shown above.
 There is no undo. Only 'yes' will be accepted to confirm.

 Enter a value:
```

Type `yes` and press **Enter**.

Expected output:

```plaintext
docker_container.nginx: Destroying...
docker_container.nginx: Destruction complete after 1s
docker_image.nginx: Destroying...
docker_image.nginx: Destru
ction complete after 0s

Destroy complete! Resources: 2 destroyed.
```

Once the `terrafrom destroy` is completed, all resources will be removed and no longer available.

---

## Next Steps

In this guide, you completed the installation of Terraform CLI, wrote a basic configuration with Docker, and a standard Terraform workflow — **init → plan → apply → destroy** — to create and destroy a Docker container. This workflow allows you to build upon this pattern, making it easier to help deploy infrastructure at scale.

To continue building your Terraform skills, we recommend exploring the following resources:

- [Terraform: Get Started (AWS, Azure, GCP)](https://developer.hashicorp.com/terraform/tutorials) — Applying this workflow with Cloud Providers.
- [Terraform Language Documentation](https://developer.hashicorp.com/terraform/language) — Helps you understand variables, outputs, modules, and state at a deeper level.
- [Terraform Cloud](https://developer.hashicorp.com/terraform/tutorials/cloud) — Helps your teams collaborate on infrastructure with remote state, policy enforcement, and team workflows.
- [HCP Terraform Free Tier](https://app.terraform.io/public/signup/account) — Get started with HCP Terraform with $500 in Free Credits
