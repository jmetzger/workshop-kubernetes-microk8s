# Exercise for macvlan 

## Prerequisite

  * You need to setup multus first (see markdown documment in this repo - using helm)

## Step 1: Setup Network Attachement 

```
mkdir -p manifests
cd manifests
mkdir -p macvlan
cd macvlan 
```

```
vi 01-network-macvlan.yml 
```

```
apiVersion: "k8s.cni.cncf.io/v1"
kind: NetworkAttachmentDefinition
metadata:
  name: ipvlan-def
spec:
  config: '{
      "cniVersion": "0.3.1",
      "type": "macvlan",
      "master": "eth1",
      "ipam": {
        "type": "host-local",
        "subnet": "192.168.200.0/24",
        "rangeStart": "192.168.200.201",
        "rangeEnd": "192.168.200.205",
        "gateway": "192.168.200.1"
      }
    }'
```
```
kubectl apply -f .
```

```
vi 02-netpod1.yml 
# on node2 
```

```
apiVersion: v1
kind: Pod
metadata:
  name: net-pod
  annotations:
    k8s.v1.cni.cncf.io/networks: ipvlan-def
spec:
  nodeName: node1
  containers:
  - name: netshoot-pod
    image: nicolaka/netshoot
    command: ["tail"]
    args: ["-f", "/dev/null"]
  terminationGracePeriodSeconds: 0
```

```
vi 03-netpod2.yml
# on node3
```

```
apiVersion: v1
kind: Pod
metadata:
  name: net-pod
  annotations:
    k8s.v1.cni.cncf.io/networks: ipvlan-def
spec:
  nodeName: node1
  containers:
  - name: netshoot-pod
    image: nicolaka/netshoot
    command: ["tail"]
    args: ["-f", "/dev/null"]
  terminationGracePeriodSeconds: 0
```

```
kubectl apply -f .
```

## Reference 

  * https://devopstales.github.io/kubernetes/multus/
