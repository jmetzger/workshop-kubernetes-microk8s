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
worker1.tln<nr>.t3isp.de
worker2.tln<nr>.t3isp.de
worker3.tln<nr>.t3isp.de
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

```

## Step 2: Setup worker1 - node (login through ssh) 

```

```

## Step 3: Setup worker2 - node (login through ssh) 

```

```

## Step 4: Setup worker3 - node (login through ssh) 

```

```
