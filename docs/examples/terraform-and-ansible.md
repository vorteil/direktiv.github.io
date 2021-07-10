---
layout: default
title: Terraform/Ansible Workflows
nav_order: 10
parent: Examples
---

# Using Terraform & Ansible in a Workflow

Terraform and Ansible are both widely used technologies in the area of automating infrastructure deployments, configurations, and more.
At first glance, it might seem daunting to incorporate these technologies into a Direktiv workflow; but it's actually fairly straightforward. 
This article seeks to explain how to structure potentially 'complex' workflows in order to support technologies that may require data that persists across multiple workflow executions.

## The Workflow

This workflow consists of only 2 states. The first state runs the `terraform` isolate, and requires access to a variable that contains the contents of a `main.tf` file. This `main.tf` file includes all of the configuration and authorisation details required to create a new virtual machine on Google Cloud Platform. The state is configured to assign the public IP address of the resulting virtual machine as `.addr`

The second state uses the `ansible` isolate, and requires access to two different variables:

- `playbook.yml`
  - Ansible playbook that will be executed to connect to the remote machine and print a 'Hello, world!' message.
- `pk.pem`
  - Private key included in the `main.tf` variable of the previous state, used by Ansible to securely connect to the remote virtual machine.

Examples of each variable are included at the end of this article.

```yaml
# This workflow uses Terraform to create an instance on 
# Google Cloud Platform, and connects to it with Ansible.
id: terraform-and-ansible
description: "This workflow uses Terraform to create an instance on Google Cloud Platform, and connects to it with Ansible."

functions:

  # Calls the standard 'terraform' isolate, providing
  # the `main.tf` file from an existing workflow variable.
  - id: terraform
    image: vorteil/terraform:v1
    files:
      - key: main.tf
        scope: workflow
        type: plain

  # Runs ansible
  - id: ansible
    image: vorteil/ansible:v1
    files:
      - key: playbook.yml
        scope: workflow
        type: plain
      - key: pk.pem
        scope: workflow

states:

  # Create a GCP instance using Terraform
  - id: run-terraform
    type: action
    action:
      secrets: ["GCP_PROJECT"]
      function: terraform
      input:
        action: "apply"
        "args-on-init": 
          - "-backend-config=address=http://localhost:8001/terraform-gcp-instance"
        variables:
          "state-name": "terraform-gcp-instance"
          project_id: jq(.secrets.GCP_PROJECT)
    transform: jq(.addr = .return.output["ip-address"].value | del(.return))
    transition: run-ansible

  # Use Ansible to connect to the instance using provided private key file
  - id: run-ansible
    type: action
    action:
      function: ansible
      input:
        playbook: playbook.yml
        privateKey: pk.pem
        args:
          - "-i"
          - "jq(.addr),"
        
```

## Variables

### main.tf

The `main.tf` file used by the `terraform` isolate ensures the creation of a Virtual Machine on Google Cloud Platform with a startup script that will save a private/public key pair to the system and ensure that it is authorised for access to the machine via SSH. To change the name of the resulting virtual machine, modify the value of the `name` field.

```tf
terraform {
    backend "http" {}
}

provider "google" {
    credentials = var.service_account_key
}

resource "google_compute_instance" "default" {
  project = var.project_id
  name         = "direktiv-terraform-ansible"
  machine_type = "n1-standard-1"
  zone         = "australia-southeast1-a"

  tags = ["direktiv", "vorteil"]

  boot_disk {
    initialize_params {
      image = "ubuntu-os-cloud/ubuntu-2004-lts"
    }
  }

  network_interface {
    network = "default"

    access_config {
      // Ephemeral IP
    }
  }

  metadata_startup_script = <<EOT
   echo <base64-encoded contents of a pem privat key> > /pk.b64
   base64 -d /pk.b64 > /pk.pem
   echo <base64-encoded contents of an rsa public key> > /ssh.b64
   base64 -d /ssh.b64 > /ssh.key
   chmod 600 /pk.pem
   eval `ssh-agent -s`
   ssh-add /pk.pem
   cat /ssh.key >> ~/.ssh/authorized_keys
EOT
}

variable "service_account_key" {
  description = "the entire contents of a service account key"
  default = <<EOT
    {
  "type": "service_account",
  "project_id": "*****",
  "private_key_id": "*****",
  "private_key": "*****",
  "client_email": "*****",
  "client_id": "*****",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/*****"
}
EOT

}
variable "project_id" {
  description = "project_id to spawn the virtual machine on"
}

output "ip-address" {
    value = google_compute_instance.default.network_interface[0].access_config[0].nat_ip
}
```

### playbook.yml

This is an incredibly basic Ansible playbook. After connecting to the remote machine, it simply prints `Hello, world!` and returns.

```yml
---
- hosts: all
  name: helloworld playbook
  gather_facts: yes
  remote_user: root
  connection: ssh
  tasks:
    - name: run helloworld logic
      ansible.builtin.debug:
        msg:
          - "Hello, world!"
```

### pk.pem

This is a PEM formatted private key file, provided to both the created virtual machine (via `main.tf`) and the `ansible` isolate. It provides a way for the `ansible` isolate to securely connect to the virtual machine via SSH.

