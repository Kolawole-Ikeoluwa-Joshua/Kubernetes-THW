# Kubernetes-THW
This repo is optimized for learning kubernetes with an end goal of understanding each task required to bootstrap a Kubernetes cluster.


## Cluster Architecture

5 VMs:

* 2 Master 
* 2 Worker
* 1 Loadbalancer


## Labs

## Labs 1:
### Prerequisites
### VM Hardware Requirements

8 GB of RAM (Preferebly 16 GB)
50 GB Disk space
### Virtual Box
Download and Install [VirtualBox](https://www.virtualbox.org/wiki/Downloads) on windows host
### Vagrant
Vagrant provides an easier way to deploy multiple virtual machines on VirtualBox more consistenlty.
Download and Install [Vagrant](https://www.vagrantup.com/) on windows host

## Labs 2:
### Provision Compute Instances

Note: You must have VirtualBox and Vagrant configured at this point

CD into vagrant directory

`cd Kubernetes-THW\vagrant`

Run Vagrant up

`vagrant up`

This does the below:

- Deploys 5 VMs - 2 Master, 2 Worker and 1 Loadbalancer with the name 'kubernetes-ha-* '
    > This is the default settings. This can be changed at the top of the Vagrant file.
    > If you choose to change these settings, please also update vagrant/ubuntu/vagrant/setup-hosts.sh
    > to add the additional hosts to the /etc/hosts default before running "vagrant up".

- Set's IP addresses in the range 192.168.5

    | VM            |  VM Name               | Purpose       | IP           | Forwarded Port   |
    | ------------  | ---------------------- |:-------------:| ------------:| ----------------:|
    | master-1      | kubernetes-ha-master-1 | Master        | 192.168.5.11 |     2711         |
    | master-2      | kubernetes-ha-master-2 | Master        | 192.168.5.12 |     2712         |
    | worker-1      | kubernetes-ha-worker-1 | Worker        | 192.168.5.21 |     2721         |
    | worker-2      | kubernetes-ha-worker-2 | Worker        | 192.168.5.22 |     2722         |
    | loadbalancer  | kubernetes-ha-lb       | LoadBalancer  | 192.168.5.30 |     2730         |

    > These are the default settings. These can be changed in the Vagrant file

- Add's a DNS entry to each of the nodes to access internet
    > DNS: 8.8.8.8

- Install's Docker on Worker nodes
- Runs the below command on all nodes to allow for network forwarding in IP Tables.
  This is required for kubernetes networking to function correctly.
    > sysctl net.bridge.bridge-nf-call-iptables=1


![Alt text](docs\images\provisioned compute instances.png?raw=true "Provisioned Instances")
