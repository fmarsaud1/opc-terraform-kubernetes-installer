[terraform]: https://terraform.io
[oci-c]: https://cloud.oracle.com/en_US/classic
[occ]: https://cloud.oracle.com/en_US/cloud-at-customer
[opc provider]: https://github.com/terraform-providers/terraform-provider-opc
[kubectl]: https://kubernetes.io/docs/tasks/tools/install-kubectl/
[ocsk]:https://docs.oracle.com/cd/E52668_01/E88884/html/pref.html

# Terraform Kubernetes Installer for Oracle Classic IaaS

## About

The Kubernetes Installer for [Oracle Classic IaaS)][oci-c] provides a Terraform-based Kubernetes installation for the
[Oracle Cloud@Customer (OCC)][occ] & [OCI-Classic (OCI-C)][oci-c] Oracle Cloud Infrastructure platforms.  
This installer utilises the [Terraform Oracle Public Cloud Provider][opc provider].
It consists of a set of [Terraform][terraform] configurations & shell scripts that are used to provision the Kubernetes control plane
in accordance with [Oracle Container Services for use with Kubernetes (OCSK)][ocsk] which is based on Kubernetes version 1.8.4, as released upstream.

## Cluster Overview

Terraform is used to _provision_ the cloud infrastructure and any required local resources for the Kubernetes cluster including:

#### OCI Infrastructure:

 - Creates single-node compute instance:
   - x1 OCPU utilised.
   - Installed OS distribution is Oracle Linux 7.2.
   - Includes associated storage and networking elements.
   - Reserves and associates a public IP address to the instance.

 - Installs Kubernetes cluster (master node):
   - Taints the node to support worker function.

#### Cluster Configuration:

Terraform uses `remote-exec` scripts to handle the instance-level _configuration_ for the instance to configure:

- Single node Kubernetes master configuration.
- Installation is in accordance with the previously referenced [Oracle Container Services for use with Kubernetes (OCSK)][ocsk]
- Kubernetes cluster version: 1.8.4.
- Kubernetes Dashboard and kube-DNS cluster add-ons.
- Optional - Monitoring and Metrics:
  - Grafana, Heapster, & InfluxDB for enhanced monitoring and metrics.
- Optional - Functions as a Service:
  - Include the [Fn](http://fnproject.io/), Fn Flow, & Fn UI server-side components. Installed via published [helm charts](#https://github.com/fnproject/fn-helm).
- Optional - Microservices Environment:
  - Include WeaveScope Microservices Dashboard and E-Commerce application. Functioning microservices e-commerce application (Socks Shop) with additional enhanced management dashboard.
- Optional - Kubernetes Ingress:
  - Include [Traefik](https://traefik.io/) Ingress and sample applications. Functioning K8s ingress & controller (L7 Load Balancer) deployed and configured to perform path based traffic steering to x3 sample microservices applications.
- Optional - Service Mesh:
  - Include [Istio](https://istio.io) Service Mesh. Functioning Istio service mesh and integrated sample microservices application (BookInfo – per istio.io).

## Prerequisites

1. Download and install [Terraform][terraform] (v0.11.3 or later). Follow the link for Hashicorp [instructions](https://www.terraform.io/intro/getting-started/install.html).
2. [Terraform OPC provider](https://www.terraform.io/docs/providers/opc/index.html#) (can be pulled automatically using terraform init
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

Use Terraform to Provision resources and stand-up k8s cluster on OCI:

```
$ terraform apply
```

At this point the configuration will prompt for the following inputs before building the cluster:

````bash
$ variable "ociUser"
$ #(input compute user account with compute_operations rights)

$ variable "ociPass"
$ #(input password for “ociUser”)

$ variable "idDomain"
$ #(input compute tenancy service instance id)

$ variable "apiEndpoint"
$ #(input compute tenancy rest endpoint url)

$ variable "containerRepoUser"
$ #(input oracle container registry username)

$ variable "containerRepoPass"
$ #(input oracle container registry password)
````

Installer will also ask the user if any of the following `environments` should be provisioned to the cluster. Enter `true` or `false` for each item accordingly:

````bash
$ Enhanced Dashboard, Monitoring and Metrics:
$ #include grafana, heapster, & influxdb..

$ microservices environment:
$ #include weavescope microservices dashboard and e-commerce application..

$ kubernetes ingress:
$ #include traefik ingress and sample applications..

$ Service Mesh:
$ #include istio service mesh, and integrated sample microservices application..

$ fn:
$ #include fn installed via published helm charts..
````

The entire build and cluster creation process is automated – no further input is required.

### Access the cluster:

The Kubernetes cluster will be running after the configuration is applied successfully, and the remote-exec scripts have completed. Typically, this takes around 15-25 minutes after `terraform apply` and will vary depending on the overall configuration, geographic location, and number of `environments` selected.

Once completed, Terraform will output the public IP address of the cluster:

````bash
$ Apply complete! Resources: 14 added, 0 changed, 0 destroyed.
$
$ Outputs:
$
$ Master_Node_Public_IPs = [
$     129.199.199.199
$]
````

Terraform will also output the Kubernetes running services and pods via tabular format.

To access Kubernetes dashboard, or any of the other web interfaces running in the cluster:
SSH tunnel to the IP address of the Kubernetes dashboard pod &/or other pods via the public/NAT IP address assigned to the compute instance. Keys are located in the directory `./ssh`.
To access the Traefik dashboard, browse to the public IP address of the instance on port 8080.

### Scale, upgrade, or delete the cluster:

During the setup process, kubeadm-setup.sh generates and outputs to stdout a token that can be used to add more nodes to the cluster. Furether instruction is available via the [Oracle Container Services for use with Kubernetes (OCSK)](https://docs.oracle.com/cd/E52668_01/E88884/html/kubernetes_install_worker.html).

## Notes
 - [Oracle Container Services for use with Kubernetes (OCSK)][ocsk]:
Oracle provides a setup and configuration script that takes advantage of the kubeadm-setup.sh cluster configuration utility. This script eases the setup on Oracle Linux including configuration of networking, firewall, proxies and the initial cluster deployment, as well as providing additional support for backup and recovery.
 - `environments:` Additional documentation, instructions and references will be included here which describe how to access and utilise each of the additional "Environments" that can be automatically provisioned to the cluster.
