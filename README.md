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

![ProvisionedInstances](https://github.com/Kolawole-Ikeoluwa-Joshua/Kubernetes-THW/blob/ff5bf11e8fbf303b2f2f74f99de56bb03c715324/docs/images/provisioned%20compute%20instances.png)

### SSH to the nodes

### 1. SSH using Vagrant

  From the directory you ran the `vagrant up` command, run `vagrant ssh <vm>` for example `vagrant ssh master-1`.
  > Note: Use VM field from the above table and not the vm name itself.

![VagrantSSH](https://github.com/Kolawole-Ikeoluwa-Joshua/Kubernetes-THW/blob/main/docs/images/master-node%20ssh%20login.png)

### 2. SSH Using SSH Client Tools

Use your favourite SSH Terminal tool (putty).

Use the above IP addresses. Username and password based SSH is disabled by default.
Vagrant generates a private key for each of these VMs. It is placed under the .vagrant folder (in the directory you ran the `vagrant up` command from) at the below path for each VM:

**Private Key Path:** `.vagrant/machines/<machine name>/virtualbox/private_key`

**Username:** `vagrant`

![SSHclient](https://github.com/Kolawole-Ikeoluwa-Joshua/Kubernetes-THW/blob/main/docs/images/ssh%20client%20login%20to%20worker%20node%201.png)



### Verify Environment

- Ensure all VMs are up

- Ensure VMs are assigned the above IP addresses
![IPconfirmation](https://github.com/Kolawole-Ikeoluwa-Joshua/Kubernetes-THW/blob/main/docs/images/master-1%20ip%20address%20confirmation.png)

- Ensure you can SSH into these VMs using the IP and private keys
![Loadbalancer](https://github.com/Kolawole-Ikeoluwa-Joshua/Kubernetes-THW/blob/main/docs/images/load%20balancer.png)

- Ensure the VMs can ping each other
![Ping](https://github.com/Kolawole-Ikeoluwa-Joshua/Kubernetes-THW/blob/main/docs/images/master-1%20pinging%20master%20node%202.png)

- Ensure the worker nodes have Docker installed on them. Version: 20.10.17
  > command `sudo docker version`
![dockerconfirmation](https://github.com/Kolawole-Ikeoluwa-Joshua/Kubernetes-THW/blob/main/docs/images/worker%202%20docker%20confirmation.png)