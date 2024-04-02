Step 1: Install Kubectl

```
curl -LO https://dl.k8s.io/release/v1.26.0/bin/linux/amd64/kubectl

chmod +x kubectl

cp kubectl /usr/bin
```
Step 2: Install K3s in the Master server

Use below command in master server to install k3s,

```
curl -sfL https://get.k3s.io | sh -

systemctl status k3s
```
Next, we need to copy the config file to use in kubectl.
```
mkdir ~/.kube

cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
```
Then check,
```
kubectl get nodes
```
K3s master node is successfully installed. next will do the worker node installation.
```

export KUBECONFIG="/etc/rancher/k3s/k3s.yaml

[root@master ~]# kubectl get node
NAME     STATUS   ROLES                  AGE     VERSION
master   Ready    control-plane,master   4d13h   v1.27.7+k3s2
```

Install Artifactory using helm:
https://jfrog.com/help/r/jfrog-installation-setup-documentation/install-artifactory-ha-with-helm?section=UUID-55f43825-c5e6-2d8b-9ee2-f65e5a29c364_N1689057907429

Install Xray using Helm:
https://jfrog.com/help/r/xray-installation-quick-start-guide-helm/database-secret
