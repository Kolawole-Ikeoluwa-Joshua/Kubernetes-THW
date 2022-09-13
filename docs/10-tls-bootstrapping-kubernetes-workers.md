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

## Pre-Requisite
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

## Configure the Binaries on the Worker node

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

## Create the Boostrap Token to be used by Nodes(Kubelets) to invoke Certificate API

For the workers(kubelet) to access the Certificates API, they need to authenticate to the kubernetes api-server first. For this we create a [Bootstrap Token](https://kubernetes.io/docs/reference/access-authn-authz/bootstrap-tokens/) to be used by the kubelet

Bootstrap Tokens take the form of a 6 character token id followed by 16 character token secret separated by a dot. Eg: abcdef.0123456789abcdef. More formally, they must match the regular expression [a-z0-9]{6}\.[a-z0-9]{16}



```
master-1$ cat > bootstrap-token-07401b.yaml <<EOF
apiVersion: v1
kind: Secret
metadata:
  # Name MUST be of form "bootstrap-token-<token id>"
  name: bootstrap-token-07401b
  namespace: kube-system

# Type MUST be 'bootstrap.kubernetes.io/token'
type: bootstrap.kubernetes.io/token
stringData:
  # Human readable description. Optional.
  description: "The default bootstrap token generated by 'kubeadm init'."

  # Token ID and secret. Required.
  token-id: 07401b
  token-secret: f395accd246ae52d

  # Expiration. Optional.
  expiration: 2023-03-10T03:22:11Z

  # Allowed usages.
  usage-bootstrap-authentication: "true"
  usage-bootstrap-signing: "true"

  # Extra groups to authenticate the token as. Must start with "system:bootstrappers:"
  auth-extra-groups: system:bootstrappers:worker
EOF


master-1$ kubectl create -f bootstrap-token-07401b.yaml

```

Things to note:
- **expiration** - make sure its set to a date in the future.
- **auth-extra-groups** - this is the group the worker nodes are part of. It must start with "system:bootstrappers:" This group does not exist already. This group is associated with this token.

Once this is created the token to be used for authentication is `07401b.f395accd246ae52d`

Reference: https://kubernetes.io/docs/reference/access-authn-authz/bootstrap-tokens/#bootstrap-token-secret-format

## Authorize workers(kubelets) to create CSR

Next we associate the group we created before to the system:node-bootstrapper ClusterRole. This ClusterRole gives the group enough permissions to bootstrap the kubelet

```
master-1$ kubectl create clusterrolebinding create-csrs-for-bootstrapping --clusterrole=system:node-bootstrapper --group=system:bootstrappers

--------------- OR ---------------

master-1$ cat > csrs-for-bootstrapping.yaml <<EOF
# enable bootstrapping nodes to create CSR
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: create-csrs-for-bootstrapping
subjects:
- kind: Group
  name: system:bootstrappers
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: system:node-bootstrapper
  apiGroup: rbac.authorization.k8s.io
EOF


master-1$ kubectl create -f csrs-for-bootstrapping.yaml

```
Reference: https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/#authorize-kubelet-to-create-csr