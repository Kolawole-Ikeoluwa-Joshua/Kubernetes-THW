# Generating the Data Encryption Config and Key

Kubernetes stores a variety of data including cluster state, application configurations, and secrets. Kubernetes supports the ability to [encrypt](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data) cluster data at rest.

In this lab you will generate an encryption key and an [encryption config](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#understanding-the-encryption-at-rest-configuration) suitable for encrypting Kubernetes Secrets.

### The Encryption Key

Generate an encryption key:

```
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```

### The Encryption Config File

Create the `encryption-config.yaml` encryption config file:

```
cat > encryption-config.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
EOF
```
![encryptionconfigfile](https://github.com/Kolawole-Ikeoluwa-Joshua/Kubernetes-THW/blob/main/docs/images/encryption%20config%20file.png)


Copy the `encryption-config.yaml` encryption config file to each controller instance:

```
for instance in master-1 master-2; do
  scp encryption-config.yaml ${instance}:~/
done
```

Move `encryption-config.yaml` encryption config file to appropriate directory.

Note: make sure the /var/lib/kubernetes/ directories are present on master-1, master-2 nodes.

on master-1 run the commands:
```
sudo mv encryption-config.yaml /var/lib/kubernetes/
```

```
for instance in master-2; do
  ssh ${instance} sudo mv encryption-config.yaml /var/lib/kubernetes/
done
```

![moveencryptionconfigs](https://github.com/Kolawole-Ikeoluwa-Joshua/Kubernetes-THW/blob/main/docs/images/move%20encryption%20config%20files.png)

Reference: https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#encrypting-your-data