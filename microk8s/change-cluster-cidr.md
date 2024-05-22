# Change cluster-cidr in microk8s - cluster 

## Node 1: Steps are a bit different  on first node 

  * You could also do it on node2 oder node3
  * It does not need to be node1 

### Step 1.1 Delete 

```
# Delete default-ippool
kubectl delete ippool default-ipv4-ippool
```

### Step 1.2 Adjust cni-config 

  * Essentially settings for daemonset calico-node 

```
# adjust cni - config
# We we will adjust the part where the calico-node
# daemonset is created
vi /var/snap/microk8s/current/args/cni-network/cni.yaml
```

```
# Search for IPV4
 - name: CALICO_IPV4POOL_CIDR
   value: "10.1.0.0/16"

# Replace with your CIDR
 - name: CALICO_IPV4POOL_CIDR
   value: "192.168.0.0/16"
```

```
# Important so a new daemonset using the new ippool, will get created 
kubectl apply -f /var/snap/microk8s/current/args/cni-network/cni.yaml  
# Check if ippool holds the new ip-range 
kubectl get ippool -o yaml 
```

## Step 1.3 change settings for kube-proxy 

```
vi /var/snap/microk8s/current/kube-proxy
```

```
# replace 
--cluster-cidr=10.1.0.0/16
# with
--cluster-cidr=192.168.0.0/16
```

### Step 1.4 Restart microk8s and check 

```
microk8s stop
microk8s start
microk8s status

# check pods
# Some system-pods might still hold old ip's 
kubectl get pods -A -o wide

# !! You need to redeploy your application pods
```

## Node 2

### Step 1.1 Adjust cni-config (only to have the same config there for later)

  * Essentially settings for daemonset calico-node 

```
# adjust cni - config
# We we will adjust the part where the calico-node
# daemonset is created
vi /var/snap/microk8s/current/args/cni-network/cni.yaml
```

```
# Search for IPV4
 - name: CALICO_IPV4POOL_CIDR
   value: "10.1.0.0/16"

# Replace with your CIDR
 - name: CALICO_IPV4POOL_CIDR
   value: "192.168.0.0/16"
```

## Step 1.2 change settings for kube-proxy 

```
vi /var/snap/microk8s/current/kube-proxy
```

```
# replace 
--cluster-cidr=10.1.0.0/16
# with
--cluster-cidr=192.168.0.0/16
```

### Step 1.3 Restart microk8s and check 

```
microk8s stop
microk8s start
microk8s status

# check pods
# Some system-pods might still hold old ip's 
kubectl get pods -A -o wide
```

## Node 3 

### Step 1.1 Adjust cni-config (only to have the same config there for later)

  * Essentially settings for daemonset calico-node 

```
# adjust cni - config
# We we will adjust the part where the calico-node
# daemonset is created
vi /var/snap/microk8s/current/args/cni-network/cni.yaml
```

```
# Search for IPV4
 - name: CALICO_IPV4POOL_CIDR
   value: "10.1.0.0/16"

# Replace with your CIDR
 - name: CALICO_IPV4POOL_CIDR
   value: "192.168.0.0/16"
```

## Step 1.2 change settings for kube-proxy 

```
vi /var/snap/microk8s/current/kube-proxy
```

```
# replace 
--cluster-cidr=10.1.0.0/16
# with
--cluster-cidr=192.168.0.0/16
```

### Step 1.3 Restart microk8s and check 

```
microk8s stop
microk8s start
microk8s status

# check pods
# Some system-pods might still hold old ip's 
kubectl get pods -A -o wide
```
