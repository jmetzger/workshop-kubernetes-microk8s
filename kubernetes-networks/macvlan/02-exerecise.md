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
      "type": "ipvlan",
      "master": "eth1",
      "mode": "l2",
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



```



## Reference 

  * https://devopstales.github.io/kubernetes/multus/
