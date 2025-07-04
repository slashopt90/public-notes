# Kubernetes on Ubuntu22.04

https://www.learnlinux.tv/how-to-build-an-awesome-kubernetes-cluster-using-proxmox-virtual-environment/

## System configuration

- Using #Ubuntu sever 22.04

```sudo apt install containerd
```
- Create the initial configuration:

```
sudo mkdir /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

For this #kubernetes cluster to work properly, we’ll need to enable SystemdCgroup within the configuration. To do that, we’ll need to edit the config we’ve just created:

```
sudo nano /etc/containerd/config.toml

```
Within that file, find the following line of text:

```
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
SystemdCgroup = true
```


- Disable swap

```
sudo swapoff -a
```

- Enable bridging

To enable bridging, we only need to edit one config file:

```
sudo nano /etc/sysctl.conf

```
Within that file, look for the following line:
Uncomment that line by removing the # symbol in front of it, which should make it look like this:

```
#net.ipv4.ip_forward=1

```


- Enable br_netfilter

The next step is to enable br_netfilter by editing yet another config file:

```
sudo nano /etc/modules-load.d/k8s.conf

```
Add the following to that file (the file should actually be empty at first):

```
br_netfilter

```

- Reboot each of your instances to ensure all of our changes so far are in place:


## Installing Kubernetes

The next step is to install the packages that are required for Kubernetes. First, we’ll add the required GPG key:

```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
sudo apt update 
sudo apt install kubeadm kubectl kubelet
```

- Initialize our Kubernetes cluster

So long as you have everything complete so far, you can initialize the Kubernetes cluster now. Be sure to customize the first IP address shown here (not the second) and also change the name to match the name of your controller.

```
sudo kubeadm init --control-plane-endpoint=172.16.250.216 --node-name controller --pod-network-cidr=10.244.0.0/16
```

After the initialization finishes, you should see at least four commands printed within the output.

### Message recieved after first node install

"our Kubernetes control-plane has initialized successfully! 
  To start using your cluster, you need to run the following as a regular user

```
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Alternatively, if you are the root user, you can run:
```
  export KUBECONFIG=/etc/kubernetes/admin.conf
```

You should now deploy a pod network to the cluster.
Run
```
kubectl apply -f [podnetwork].yaml
```
with one of the options listed at:
    https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

```
  kubeadm join 192.168.10.66:6443 --token 2be5af.zxztnj9vmgvnzmat \
	--discovery-token-ca-cert-hash sha256:803dce1dcdbaa60dfd065300668fefe8037a176a2e03fbb44193e73ba83fdf29 \
	--control-plane 
```

Then you can join any number of worker nodes by running the following on each as root:

```
kubeadm join 192.168.10.66:6443 --token 2be5af.zxztnj9vmgvnzmat \
	--discovery-token-ca-cert-hash sha256:803dce1dcdbaa60dfd065300668fefe8037a176a2e03fbb44193e73ba83fdf29 
```
"

## Installing MetalLB 

### Protocol: layer 2

> see https://metallb.universe.tf/configuration/

configuration yaml 

```
  apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.10.240-192.168.10.254
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: example
  namespace: metallb-system
```

specificare un range di indirizzo ip gestito dal router!


## Install ingress-nginx-controller

using helm

```
  helm install ingress-nginx ingress-nginx/ingress-nginx --version 4.10.0 -n
ingress-nginx --create-namespace
```

IP 192.168.10.241


## Install Longhorn
- open-iscsi must be installed on nodes

```
helm repo add longhorn https://charts.longhorn.io
helm repo update
kubectl create namespace longhorn-system
helm install longhorn longhorn/longhorn --namespace longhorn-system
```
### Setting up ingress with basic auth

creates user and password file, creates the secret
```
USER=admin; PASSWORD=Passw99rd; echo "${USER}:$(openssl passwd -stdin -apr1 <<< ${PASSWORD})" >> auth
kubectl -n longhorn-system create secret generic basic-auth --from-file=auth
```

patch the frontend service to have an external IP from the load balancer
```
  kubectl patch svc longhorn-frontend -n longhorn-system -p '{"spec": {"type": "LoadBalancer"}}'
```

#### Creating certificates
using #mkcert

```
  mkcert -install
  mkcert longhorn-ui.k8s.fga.local
```

this will creates two files: 
  - longhorn-ui.k8s.fga.local-key.pem  -> certificate key
  - longhorn-ui.k8s.fga.local.pem -> certificate

creates the secret for this certificates
```
kubectl create secret tls -n longhorn-system longhorn-ingress-secret --cert=longhorn-ui.k8s.fga.local.pem --key=longhorn-ui.k8s.fga.local-key.pem
 
```

create the manifest
```
  apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: longhorn-ingress
  namespace: longhorn-system
  annotations:
    # type of authentication
    nginx.ingress.kubernetes.io/auth-type: basic
    # prevent the controller from redirecting (308) to HTTPS
    nginx.ingress.kubernetes.io/ssl-redirect: 'false'
    # name of the secret that contains the user/password definitions
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    # message to display with an appropriate context why the authentication is required
    nginx.ingress.kubernetes.io/auth-realm: 'Authentication Required '
    # custom max body size for file uploading like backing image uploading
    nginx.ingress.kubernetes.io/proxy-body-size: 10000m
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - longhorn-ui.k8s.fga.local
    secretName: longhorn-ingress-secret 
  rules:
    - host: longhorn-ui.k8s.fga.local
      http:
        paths:
        - pathType: Prefix
          path: "/"
          backend:
            service:
              name: longhorn-frontend
              port:
                number: 80
```

apply the manifest

```
  kubectl patch svc longhorn-frontend -n longhorn-system -p '{"spec": {"type": "LoadBalancer"}}'
```
