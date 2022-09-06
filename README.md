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



### Troubleshooting Tips

1. If any of the VMs failed to provision, or is not configured correct, delete the vm using the command:

`vagrant destroy <vm>`

Then reprovision. Only the missing VMs will be re-provisioned

`vagrant up`


Sometimes the delete does not delete the folder created for the vm and throws the below error.

VirtualBox error:

    VBoxManage.exe: error: Could not rename the directory 'D:\VirtualBox VMs\ubuntu-bionic-18.04-cloudimg-20190122_1552891552601_76806' to 'D:\VirtualBox VMs\kubernetes-ha-worker-2' to save the settings file (VERR_ALREADY_EXISTS)
    VBoxManage.exe: error: Details: code E_FAIL (0x80004005), component SessionMachine, interface IMachine, callee IUnknown
    VBoxManage.exe: error: Context: "SaveSettings()" at line 3105 of file VBoxManageModifyVM.cpp

In such cases delete the VM, then delete the VM folder and then re-provision

`vagrant destroy <vm>`

`rmdir "<path-to-vm-folder>\kubernetes-ha-worker-2"`

`vagrant up`



## Labs 3:

### Installing the Client Tools
master-1 node will perform administrative tasks, such as creating certificates, kubeconfig files and distributing them to the different VMs.

Note: make sure that system is able to access all the provisioned VMs through SSH to copy files over.

## Access all VMs

Generate Key Pair on master-1 node
`$ssh-keygen`

Leave all settings to default.

View the generated public key ID at:

```
$cat .ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQD......8+08b vagrant@master-1
```

Move public key of master to all other VMs

```
$cat >> ~/.ssh/authorized_keys <<EOF
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQD......8+08b vagrant@master-1
EOF
```
try to SSH into all nodes from master-1 node

![Adminsetup](https://github.com/Kolawole-Ikeoluwa-Joshua/Kubernetes-THW/blob/main/docs/images/setting%20up%20master-1%20for%20admin%20purpose.png)



### Install kubectl

The [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl). command line utility is used to interact with the Kubernetes API Server. Download and install `kubectl` from the official release binaries:

Reference: [https://kubernetes.io/docs/tasks/tools/install-kubectl/](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

### Linux

```
wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kubectl
```

```
chmod +x kubectl
```

```
sudo mv kubectl /usr/local/bin/
```

### Verification

Verify `kubectl` version 1.13.0 or higher is installed:

```
kubectl version --client
```

> output

```
Client Version: version.Info{Major:"1", Minor:"13", GitVersion:"v1.13.0", GitCommit:"ddf47ac13c1a9483ea035a79cd7c10005ff21a6d", GitTreeState:"clean", BuildDate:"2018-12-03T21:04:45Z", GoVersion:"go1.11.2", Compiler:"gc", Platform:"linux/amd64"}
```


## Labs 4:

### Provisioning a CA and Generating TLS Certificates

In this lab you will provision a PKI Infrastructure using the popular openssl tool, then use it to bootstrap a Certificate Authority, and generate TLS certificates for the following components: etcd, kube-apiserver, kube-controller-manager, kube-scheduler, kubelet, and kube-proxy.

Log into master node-1, then provision a Certificate Authority that can be used to generate additional TLS certificates.

Create a CA certificate, then generate a Certificate Signing Request and use it to create a private key:

```
# Create private key for CA
openssl genrsa -out ca.key 2048

# Comment line starting with RANDFILE in /etc/ssl/openssl.cnf definition to avoid permission issues
sudo sed -i '0,/RANDFILE/{s/RANDFILE/\#&/}' /etc/ssl/openssl.cnf

# Create CSR using the private key
openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr

# Self sign the csr using its own private key
openssl x509 -req -in ca.csr -signkey ca.key -CAcreateserial  -out ca.crt -days 1000
```
Results:

```
ca.crt
ca.key
```
![CA](https://github.com/Kolawole-Ikeoluwa-Joshua/Kubernetes-THW/blob/main/docs/images/create%20CA,%20CSR,%20private%20key.png)


