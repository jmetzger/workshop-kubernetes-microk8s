# Installation of Kubernetes Cluster API

## Prerequisites 

  * You need to have a Kubernetes Cluster running (this will be the management cluster) 
    * Within that you will you cluster api 
    * This could be something like kind, rancherdesktop.io
    * And of course also a cluster on premise 

## Step 1: Create Management Cluster 

### Step 1a: Install clusterctl 

```
# Install rancherdesktop.io 
# You are able to use it on windows (that's what we do now, and install 

# Install clusterctl in wsl -> Ubuntu 
sudo su -
cd /usr/src 
curl -L https://github.com/kubernetes-sigs/cluster-api/releases/download/v1.4.2/clusterctl-linux-amd64 -o clusterctl
sudo install -o root -g root -m 0755 clusterctl /usr/local/bin/clusterctl
clusterctl version 
```

  * Reference gist: https://gist.github.com/vfarcic/d8113b6f149583e1cf1614d76f2a4182
  * https://cluster-api.sigs.k8s.io/user/quick-start.html#install-clusterctl

### Step 1b: Set env variables for digitalocean 

```
export DIGITALOCEAN_ACCESS_TOKEN=[...] # Replace with your token here 

```


### Step 1c: Create kubernetes snapshot to be used for Kubernetes Control Plane and workers 

``` 
# can be done as unprivileged user !!! 
export PATH=$PATH:~/.local/bin
sudo apt update
apt install -y jq zip 
sudo git clone https://github.com/kubernetes-sigs/image-builder
cd image-builder/images/capi
cat Makefile
# Size of machine will always be 1gb and 1vcpu created in NYC1 
make build-do-ubuntu-2004
```

### Step 1d: Install doctl optional

```
# works in most cases on wsl, but only if snap is working properly 
# snap install doctl 
# if not do -> this 

cd ~
wget https://github.com/digitalocean/doctl/releases/download/v1.94.0/doctl-1.94.0-linux-amd64.tar.gz
tar xf ~/doctl-1.94.0-linux-amd64.tar.gz
sudo mv ~/doctl /usr/local/bin

# now authenticate 
doctl auth init --access-token ${DIGITALOCEAN_ACCESS_TOKEN}

```

### Step 1e: Set env for to create worker cluster with controlplane and workers 

```
# control the datacenter - default nyc1 
export DO_REGION=fra1
# control size of machines
# default 1vcpu-1gb 
export DO_CONTROL_PLANE_MACHINE_TYPE=s-2vcpu-2gb
export DO_NODE_MACHINE_TYPE=s-2vcpu-2gb
# needed to set up the api provider 
export DO_B64ENCODED_CREDENTIALS="$(\
    echo -n "$DIGITALOCEAN_ACCESS_TOKEN" \
    | base64 \
    | tr -d '\n')"
    
# get the snapshot id / get the right id 
doctl compute image list-user
# e.g. 
# 132627725

export DO_CONTROL_PLANE_MACHINE_IMAGE=132627725
export DO_NODE_MACHINE_IMAGE=132627725

```

### Step 1f: Setup cluster and api-provider 

```
## In our case it sets up the management cluster on rancher 
## to be used for kubernetes 
cd ../../../

clusterctl init \
    --infrastructure digitalocean

```

### Step 1g: Generate the yaml scripts for both control plane and workers 

```
# it looks there will be a fingerprint to be used, which chooses the ssh-key to be used
# to connect to the machines
# look for all the ssh-key like so:
doctl compute ssh-key list 

# So we choose one from the list 
export DO_SSH_KEY_FINGERPRINT=[...]

# Kuberentes must be the same version as you created the snapshots for do
# to be used for digitalocean -> creating a cluster there
clusterctl generate cluster devops-toolkit \
    --infrastructure digitalocean \
    --target-namespace infra \
    --kubernetes-version v1.24.11 \
    --control-plane-machine-count 3 \
    --worker-machine-count 3 \
    | tee cluster.yaml
    
kubectl create namespace infra

kubectl apply --filename cluster.yaml
```

### Step 1h: Let us observe - In your management cluster ...

``` 
### ... you will find more information
kubectl --namespace infra \
    get clusters

kubectl --namespace infra \
    get machinedeployments

kubectl --namespace infra \
    get kubeadmcontrolplane

kubectl --namespace infra \
    describe kubeadmcontrolplane \
    devops-toolkit-control-plane
```

### Step 1i: Setup calico 

```


```
