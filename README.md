## **Deploy-Consul-Cluster-Using-Terraform-Modules-On-AWS** 

********Prerequisite*********
*   1. Terraform            *
*   2. Packer               *
*   3. AWS Account          *
*   4. AWS CLI              *
*   5. Git hub              *
*****************************

This repo is based on Terraform's resuable module registry and suitable for any custom requirement, it also include an example repo for highly secure and encrypted module with custom parameters for consul cluster deployment.    


## What's a Module?

A Module is a canonical, reusable, best-practices definition for how to run a single piece of infrastructure, such
as a database or server cluster. Each Module is created using [Terraform](https://www.terraform.io/), and
includes automated tests, examples, and documentation. It is maintained both by the open source community and
companies that provide commercial support.

Instead of figuring out the details of how to run a piece of infrastructure from scratch, you can reuse
existing code that has been proven in production. And instead of maintaining all that infrastructure code yourself,
you can leverage the work of the Module community to pick up infrastructure improvements through
a version number bump.



 **Consul AWS Module

This repo contains a set of modules in the [modules folder](https://github.com/nakulbhati/Automated-Deployment-Of-Consul-Cluster-Using-Terraform/tree/master/modules) for deploying a [Consul](https://www.consul.io/) cluster on
[AWS](https://aws.amazon.com/) using [Terraform](https://www.terraform.io/). 

Consul is a distributed, highly-available tool that you can use for service discovery and key/value storage. A Consul cluster typically includes a small number of server nodes, which are responsible for being part of the [consensus
quorum](https://www.consul.io/docs/internals/consensus.html), and a larger number of client nodes, which you typically run alongside your apps:

![Consul architecture](https://github.com/hashicorp/terraform-aws-consul/blob/master/_docs/architecture.png?raw=true)



This repo has the following folder structure:

* [modules](https://github.com/nakulbhati/Automated-Deployment-Of-Consul-Cluster-Using-Terraform/tree/master/modules): This folder contains several standalone, reusable, production-grade modules that you can use to deploy Consul.
* [examples](https://github.com/nakulbhati/Automated-Deployment-Of-Consul-Cluster-Using-Terraform/tree/master/examples): This folder shows examples of different ways to combine the modules in the `modules` folder to deploy Consul.
* [root folder](https://github.com/nakulbhati/Automated-Deployment-Of-Consul-Cluster-Using-Terraform/tree/master/examples/root-example): The root folder is *an example* of how to use the [consul-cluster module](https://github.com/nakulbhati/Automated-Deployment-Of-Consul-Cluster-Using-Terraform/tree/master/modules/consul-cluster)

  module to deploy a [Consul](https://www.consul.io/) cluster in [AWS](https://aws.amazon.com/). The Terraform Registry requires the root of every repo to contain Terraform code, so I've put one of the examples there. 

To deploy Consul servers for production using this repo:

1. Create a Consul AMI using a Packer template that references the [install-consul module](https://github.com/nakulbhati/Automated-Deployment-Of-Consul-Cluster-Using-Terraform/tree/master/modules/install-consul).
   Here is an [example Packer template](https://github.com/nakulbhati/Automated-Deployment-Of-Consul-Cluster-Using-Terraform/tree/master/examples/consul-ami).


2. Deploy that AMI across an Auto Scaling Group using the Terraform [consul-cluster module](https://github.com/hashicorp/terraform-aws-consul/tree/master/modules/consul-cluster)
   and execute the [run-consul script](https://github.com/hashicorp/terraform-aws-consul/tree/master/modules/run-consul) with the `--server` flag during boot on each
   Instance in the Auto Scaling Group to form the Consul cluster. Here is [an example Terraform
   configuration](https://github.com/hashicorp/terraform-aws-consul/tree/master/examples/root-example#quick-start) to provision a Consul cluster.

To deploy Consul clients for production using this repo:

1. Use the [install-consul module](https://github.com/nakulbhati/Automated-Deployment-Of-Consul-Cluster-Using-Terraform/tree/master/modules/install-consul) to install Consul alongside your application code.
1. Before booting your app, execute the [run-consul script](hhttps://github.com/nakulbhati/Automated-Deployment-Of-Consul-Cluster-Using-Terraform/tree/master/modules/run-consul) with `--client` flag.
1. Your app can now use the local Consul agent for service discovery and key/value storage.
1. Use the [install-dnsmasq module](https://github.com/nakulbhati/Automated-Deployment-Of-Consul-Cluster-Using-Terraform/tree/master/install-dnsmasq) to configure Consul as the DNS for a
   specific domain (e.g. `.consul`) so that URLs such as `foo.service.consul` resolve automatically to the IP
   address(es) for a service `foo` registered in Consul (all other domain names will be continue to resolve using the
   default resolver on the OS).


**Note: From Consul 0.7.1 new configuration options were added to allow bootstrapping by automatically discovering AWS instances with a given tag key/value at startup. This is game changing because the hard work is done for you - all you need to do is ensure all of the Consul server instances share a tag and are able to communicate with one another.
