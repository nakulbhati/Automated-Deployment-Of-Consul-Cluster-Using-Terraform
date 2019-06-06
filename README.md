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


Note: From Consul 0.7.1 new configuration options were added to allow bootstrapping by automatically discovering AWS instances with a given tag key/value at startup. This is game changing because the hard work is done for you - all you need to do is ensure all of the Consul server instances share a tag and are able to communicate with one another.


**************************************************************************************************************

## Ongoing maintenance tasks to carry out

**1. Memory usage

Metric Name	        Description
mem.total	          Total amount of physical memory (RAM) available on the server.
mem.used_percent	  Percentage of physical memory in use.
swap.used_percent	  Percentage of swap space in use.


Why they're important: Consul keeps all of its data in memory. If Consul consumes all available memory, it will crash. You should also monitor total available RAM to make sure some RAM is available for other processes, and swap usage should remain at 0% for best performance.

What to look for: If mem.used_percent is over 90%, or if swap.used_percent is greater than 0.

***********************************************************************************************

**2. File descriptors

Metric Name	                Description
linux_sysctl_fs.file-nr	    Number of file handles being used across all processes on the host.
linux_sysctl_fs.file-max	  Total number of available file handles.


Why it's important:Practically anything Consul does -- receiving a connection from another host, sending data between servers, writing snapshots to disk -- requires a file descriptor handle. If Consul runs out of handles, it will stop accepting connections. See the Consul FAQ for more details.

By default, process and kernel limits are fairly conservative. You will want to increase these beyond the defaults.

What to look for: If file-nr exceeds 80% of file-max


************************************************************************************************

**3. CPU usage


Metric Name	          Description
cpu.user_cpu	        Percentage of CPU being used by user processes (such as Consul).
cpu.iowait_cpu	      Percentage of CPU time spent waiting for I/O tasks to complete.


Why they're important: Consul is not particularly demanding of CPU time, but a spike in CPU usage might indicate too many operations taking place at once, and iowait_cpu is critical -- it means Consul is waiting for data to be written to disk, a sign that Raft might be writing snapshots to disk too often.


What to look for: if cpu.iowait_cpu greater than 10%.

*************************************************************************************************


**4. Network activity - Bytes Recived


Metric Name	          Description
net.bytes_recv	      Bytes received on each network interface.
net.bytes_sent	      Bytes transmitted on each network interface.


Why they're important: A sudden spike in network traffic to Consul might be the result of a misconfigured application client causing too many requests to Consul. This is the raw data from the system, rather than a specific Consul metric.


What to look for: Sudden large changes to the net metrics (greater than 50% deviation from baseline).

NOTE: The net metrics are counters, so in order to calculate rates (such as bytes/second), you will need to apply a function such as non_negative_difference.

**************************************************************************************


**5. Disk activity

Metric Name	            Description
diskio.read_bytes	      Bytes read from each block device.
diskio.write_bytes	    Bytes written to each block device.

Why they're important: If the Consul host is writing a lot of data to disk, such as under high volume workloads, there may be frequent major I/O spikes during leader elections. This is because under heavy load, Consul is checkpointing Raft snapshots to disk frequently.

It may also be caused by Consul having debug/trace logging enabled in production, which can impact performance.

Too much disk I/O can cause the rest of the system to slow down or become unavailable, as the kernel spends all its time waiting for I/O to complete.

What to look for: Sudden large changes to the diskio metrics (greater than 50% deviation from baseline, or more than 3 standard deviations from baseline).

NOTE: The diskio metrics are counters, so in order to calculate rates (such as bytes/second), you will need to apply a function such as non_negative_difference.
