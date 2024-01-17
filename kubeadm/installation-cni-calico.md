# Installation mit kubeadm (CNI: calico) 

## Version 

  * Ubuntu 20.04 LTS 

## Done for you 

  * Servers are setup:
    * ssh-running
    * kubeadm, kubelet, kubectl installed
    * containerd - runtime installed 

## Prerequisites 

  * 4 Servers setup and reachable through ssh.
  * user: 11trainingdo
  * pass: PLEASE ask your instructor 


```
# Important - Servers are not reachable through
# Domain !! Only IP. 
controlplane.tln<nr>.t3isp.de 
worker1.tln<nr>.do.t3isp.de
worker2.tln<nr>.do.t3isp.de
worker3.tln<nr>.do.t3isp.de
```

## Step 1: Setup controlnode (login through ssh) 

```
# This CIDR is the recommendation for calico
# Other CNI's might be different 
CLUSTER_CIDR="192.168.0.0/16"

kubeadm init --pod-network-cidr=$CLUSTER_CIDR && \
  mkdir -p /root/.kube && \
  cp -i /etc/kubernetes/admin.conf /root/.kube/config && \
  chown $(id -u):$(id -g) /root/.kube/config
```

```
# Copy output of join (needed for workers) 
# e.g. 
kubeadm join 159.89.99.35:6443 --token rpylp0.rdphpzbavdyx3llz \
        --discovery-token-ca-cert-hash sha256:05d42f2c051a974a27577270e09c77602eeec85523b1815378b815b64cb99932
```

## Step 2: Setup worker1 - node (login through ssh) 

```
# use join command from Step 1:
kubeadm join 159.89.99.35:6443 --token rpylp0.rdphpzbavdyx3llz \
        --discovery-token-ca-cert-hash sha256:05d42f2c051a974a27577270e09c77602eeec85523b1815378b815b64cb99932
```

## Step 3: Setup worker2 - node (login through ssh) 

```
# use join command from Step 1:
kubeadm join 159.89.99.35:6443 --token rpylp0.rdphpzbavdyx3llz \
        --discovery-token-ca-cert-hash sha256:05d42f2c051a974a27577270e09c77602eeec85523b1815378b815b64cb99932
```

## Step 4: Setup worker3 - node (login through ssh) 

```
# use join command from Step 1:
kubeadm join 159.89.99.35:6443 --token rpylp0.rdphpzbavdyx3llz \
        --discovery-token-ca-cert-hash sha256:05d42f2c051a974a27577270e09c77602eeec85523b1815378b815b64cb99932
```

## Step 5: CNI-Setup (calico) on controlnode (login through ssh) 

```
kubectl get nodes 
```

```
# Output
root@controlplane:~# kubectl get nodes
NAME           STATUS     ROLES           AGE     VERSION
controlplane   NotReady   control-plane   6m27s   v1.28.6
worker1        NotReady   <none>          3m18s   v1.28.6
worker2        NotReady   <none>          2m10s   v1.28.6
worker3        NotReady   <none>          60s     v1.28.6
```
