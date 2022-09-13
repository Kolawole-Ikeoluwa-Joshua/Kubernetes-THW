# TLS Bootstrapping Worker Nodes

In the previous step we configured a worker node by
- Creating a set of key pairs for the worker node by ourself
- Getting them signed by the CA by ourself
- Creating a kube-config file using this certificate by ourself
- Everytime the certificate expires we must follow the same process of updating the certificate by ourself

This is not a practical approach when you have 1000s of nodes in the cluster, and nodes dynamically being added and removed from the cluster.  With TLS boostrapping:

- The Nodes can generate certificate key pairs by themselves
- The Nodes can generate certificate signing request by themselves
- The Nodes can submit the certificate signing request to the Kubernetes CA (Using the Certificates API)
- The Nodes can retrieve the signed certificate from the Kubernetes CA
- The Nodes can generate a kube-config file using this certificate by themselves
- The Nodes can start and join the cluster by themselves
- The Nodes can request new certificates via a CSR, but the CSR must be manually approved by a cluster administrator

In Kubernetes 1.11 a patch was merged to require administrator or Controller approval of node serving CSRs for security reasons.

Reference: https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/#certificate-rotation

So let's get started!

# What is required for TLS Bootstrapping

**Certificates API:** The Certificate API (as discussed in the lecture) provides a set of APIs on Kubernetes that can help us manage certificates (Create CSR, Get them signed by CA, Retrieve signed certificate etc). The worker nodes (kubelets) have the ability to use this API to get certificates signed by the Kubernetes CA.

### Pre-Requisite
Use the process status command on linux to verify the following:

`ps -aux | grep kube-api`

**kube-apiserver** - Ensure bootstrap token based authentication is enabled on the kube-apiserver.

`--enable-bootstrap-token-auth=true`

**kube-controller-manager** - The certificate requests are signed by the kube-controller-manager ultimately. The kube-controller-manager requires the CA Certificate and Key to perform these operations.

`ps -aux | grep kube-controller-manager`

```
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.crt \\
  --cluster-signing-key-file=/var/lib/kubernetes/ca.key
```

> Note: We have already configured these in our setup in this course

Copy the ca certificate to the worker-2 node:
```
scp ca.crt worker-2:~/
```

### Configure the Binaries on the Worker node

### Download and Install Worker Binaries

```
worker-2$ wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kubelet
```

Reference: https://kubernetes.io/docs/setup/release/#node-binaries

Create the installation directories:

```
worker-2$ sudo mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

Install the worker binaries:

```
{
  chmod +x kubectl kube-proxy kubelet
  sudo mv kubectl kube-proxy kubelet /usr/local/bin/
}
```
### Move the ca certificate

`worker-2$ sudo mv ca.crt /var/lib/kubernetes/