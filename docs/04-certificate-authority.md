## Labs 4:

### Provisioning a CA and Generating TLS Certificates

In this lab you will provision a PKI Infrastructure using the popular openssl tool, then use it to bootstrap a Certificate Authority, and generate TLS certificates for the following components: etcd, kube-apiserver, kube-controller-manager, kube-scheduler, kubelet, and kube-proxy.

### Certificate Authority
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

Reference : https://kubernetes.io/docs/concepts/cluster-administration/certificates/#openssl

The ca.crt is the Kubernetes Certificate Authority certificate and ca.key is the Kubernetes Certificate Authority private key.
You will use the ca.crt file in many places, so it will be copied to many places.
The ca.key is used by the CA for signing certificates. And it should be securely stored. In this case our master node(s) is our CA server as well, so we will store it on master node(s). There is not need to copy this file to elsewhere.



