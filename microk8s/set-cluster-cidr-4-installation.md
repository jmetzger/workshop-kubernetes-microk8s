# Using launch configuration for installation - cluster/service - cidr (from 1.28+) 

## Version without adding node 

### Step 1: Setup /etc/microk8s.yaml on node 

#### Version 1: Without joining (from microk8s 1.28+)

  * Nice, because it sets all the right places 

```
---
version: 0.2.0
extraCNIEnv:
  IPv4_CLUSTER_CIDR: "172.18.0.0/16"
  IPv4_SERVICE_CIDR: "172.17.0.0/24"
extraSANs:
  - 172.17.0.1
addons:
  - name: dns
  - name: rbac 
```

#### Version 2: With persistent cluster token 

  * Always use the same token

```
---
version: 0.2.0
persistentClusterToken: "a74cddf30d2408d49fcd748a26021c6a"
join:
# adjust x to your ip
  url: "10.135.0.x:25000/a74cddf30d2408d49fcd748a26021c6a"   
extraCNIEnv:
  IPv4_CLUSTER_CIDR: "172.18.0.0/16"
  IPv4_SERVICE_CIDR: "172.17.0.0/24"
extraSANs:
  - 172.17.0.1
addons:
  - name: dns
  - name: rbac
```


### Step 2: Installieren von microk8s 

  * Wenn einer fehler in der config ist, installiert er nicht ! (Das ist gut)

```
snap install microk8s --classic 
```

## Reference: 

 *  https://github.com/canonical/microk8s-cluster-agent/blob/main/pkg/k8sinit/testdata/schema/full.yaml
