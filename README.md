[terraform]: https://terraform.io
[oci-c]: https://cloud.oracle.com/en_US/classic
[occ]: https://cloud.oracle.com/en_US/cloud-at-customer
[opc provider]: https://github.com/terraform-providers/terraform-provider-opc

# Terraform LAMP Installer for Oracle Classic IaaS
![readme md_logo_v0 01](https://user-images.githubusercontent.com/36317955/36942468-8f8e3942-1fc6-11e8-9d73-8acd647ae0f4.png)
## About

The LAMP Installer for [Oracle Classic IaaS][oci-c] provides a Terraform-based LAMP installation for the
[Oracle Cloud@Customer (OCC)][occ] & [OCI-Classic (OCI-C)][oci-c] Oracle Cloud Infrastructure platforms.  

This installer utilises the [Terraform Oracle Public Cloud Provider][opc provider].
It consists of a set of [Terraform][terraform] configurations & shell scripts which are used to provision a Two-Tier LAMP Stack, and associated Management Network. The softwre elements of the LAMP stack are based on open source technologies.

## Solution Overview

Terraform is used to _provision_ the cloud infrastructure and any required local resources for the LAMP Stack including:

#### OCI Infrastructure:

 - 3 IP Networks:
   - **IP Network 1: _MANAGEMENT (public)_**
     - x1 Bastion/NAT Gateway:
       - Allows SSH inbound from public internet to provision & admin the environment.
       - Is configured as NAT Gateway for outbound traffic originating from private networks.

   - **IP Network 2: _DATABASE (private)_**
     - X1 MySQL Server:
       - Database server uses NAT Gateway for outbound internet access (i.e. to install packages from public repositories).

   - **IP Network 3: _WEB FRONT-END (public)_**
     - X1 Web Server:
       - Apache, PHP, MyPHPAdmin (connected to database server), Apache Server-Info, Apache Server-Status.

Terraform uses `remote-exec` provisioner to handle the instance-level _configuration_ for the instances to provision out the Apache, MySQL, and Internet Gateway functionality.

## Prerequisites

1. Download and install [Terraform][terraform] (v0.11.3 or later). Follow the link for Hashicorp [instructions](https://www.terraform.io/intro/getting-started/install.html).
2. [Terraform OPC provider](https://www.terraform.io/docs/providers/opc/index.html#) (can be pulled automatically using `terraform init`
directive once Terraform is configured).

## Quick start
### Deploy the cluster:

Initialize Terraform:

```
$ terraform init
``` 

View what Terraform plans do before actually doing it:

```
$ terraform plan
```

Use Terraform to provision resources and stand-up application components OCI:

```
$ terraform apply
```

At this point the configuration will prompt for the following inputs before building the environment:

````bash
$ variable "ociUser"
$ #(input compute user account with compute_operations rights)

$ variable "ociPass"
$ #(input password for “ociUser”)

$ variable "idDomain"
$ #(input compute tenancy service instance id)

$ variable "apiEndpoint"
$ #(input compute tenancy rest endpoint url)
````

The entire build and LAMP configuration process is automated – no further input is required.

### Access the environment:

The LAMP stack will be running after the configuration is applied successfully, and the remote-exec scripts have completed. Typically, this takes around 7-9 minutes after `terraform apply`.

Once completed, Terraform will output the public IP addresses of the environment:

````bash
$ Apply complete! Resources: 23 added, 0 changed, 0 destroyed.
$ 
$ Outputs:
$ 
$ Application_Instance_Private_IPs = [
$     10.2.0.10
$ ]
$ Application_Instance_Public_IPs = [
$     140.86.0.2
$ ]
$ Database_Instance_Private_IPs = [
$     10.3.0.10
$ ]
$ Management_Instance_Public_IPs = [
$     140.86.0.17
$ ]
````

To access MySQL admin dashboard, or any of the other web interfaces running in the stack, browse to the public IP address of the application host on port 80 - followed by any of the paths as described:

  - `/`  
    Provides the default Apache landing page
  - `/phpMyAdmin`  
    Provides access to the MySQL admin tool. Log in as User: root Pass: password. It will connect automatically to the database server.
  - `/server-status`  
    Server status reports generated by Apache mod_status
  - `/server-info`  
    Apache server configuration report

_Keys are provided for simplicity only, for long running deployments it is recommended that you replace the provided keys prior to deployment._

## Notes
 - Future: Include option to choose either IaaS or PaaS database engine at initialisation phase.
