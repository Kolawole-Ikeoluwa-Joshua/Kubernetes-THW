# Generating Kubernetes Configuration Files for Authentication

In this lab you will generate [Kubernetes configuration files](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/), also known as kubeconfigs, which enable Kubernetes clients to locate and authenticate to the Kubernetes API Servers.

### Client Authentication Configs

In this section you will generate kubeconfig files for the `controller manager`, `kube-proxy`, `scheduler` clients and the `admin` user.

### Kubernetes Public IP Address

Note: Each kubeconfig requires a Kubernetes API Server to connect to. 

To support high availability the IP address assigned to the load balancer `192.168.5.30`, will be used for client components (`kube-proxy` & `admin` user) that live outside the master node as the kube-api server.  

First set load balancer address:

```
LOADBALANCER_ADDRESS=192.168.5.30
```
### The kube-proxy Kubernetes Configuration File

Generate a kubeconfig file for the `kube-proxy` service:

```
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-credentials system:kube-proxy \
    --client-certificate=kube-proxy.crt \
    --client-key=kube-proxy.key \
    --embed-certs=true \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
}
```

Results:

```
kube-proxy.kubeconfig
```
![kube-proxy-kubeconfig](https://github.com/Kolawole-Ikeoluwa-Joshua/Kubernetes-THW/blob/main/docs/images/kube-proxy%20config%20file.png)

Reference docs for kube-proxy [here](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/)