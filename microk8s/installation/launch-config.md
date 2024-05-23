# Using a launch-config for microk8s 

```
Since microk8s 1.27 it is possible to use
a launchconfig
```

## Default LaunchConfig being used:

  * https://github.com/canonical/microk8s/blob/master/microk8s-resources/microk8s.default.yaml


```
# Place were this can be found on an installed/deployed microk8s system
/var/snap/microk8s/current/microk8s.default.yaml
```

```
# Default launch configuration for MicroK8s.
---
version: 0.1.0
extraKubeletArgs:
  --cluster-domain: cluster.local
  --cluster-dns: 10.152.183.10
addons:
  - name: dns
```


## How to overwrite it:

```
# Looks like it is possible by create a .microk8s.default.yaml - file
# under /var/snap/microk8s/common
#
# this folder will no be there by default
mkdir -p /var/snap/microk8s/common
# Attention: microk8s-config.yaml is not there, you have to create it on your own 
sudo cp microk8s-config.yaml /var/snap/microk8s/common/.microk8s.yaml

```

  * Reference: https://microk8s.io/docs/add-launch-config

## Use it to changed cluster-cidr 

```
# Part 1: from microk8s 1.28 and newer
---
version: 0.2.0
extraCNIEnv:
  IPv4_CLUSTER_CIDR: "10.2.0.0/16"
  IPv4_SERVICE_CIDR: "10.94.0.0/24"
extraSANs:
  - 10.94.0.1
addons:
  - name: dns
```

  * In my opinion it also needs to be added to the kube-proxy

```
# 'extraKubeProxyArgs' is extra arguments to add to the local node kube-proxy.
# Set a value to null to remove it from the arguments.
extraKubeProxyArgs:
  --cluster-cidr: 10.1.0.0/16
```


## Reference: 

  * https://microk8s.io/docs/add-launch-config



