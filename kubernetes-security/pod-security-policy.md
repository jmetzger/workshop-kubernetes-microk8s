# Pod Security Policy (PSP)  

## General 

  * PodSecurity is an eine Rolle gebunden (clusterrole)
  * Deprecated in 1.21 removed in 1.25
  * From 1.25 on please use PSA (Pod Security Admission) instead 

## Prerequisites 

  * We should have a running Cluster of 1.22/1.23


## Walkthrough 

### Step 1: Create Digitalocean microk8s 1-node - cluster, with this cloud-init-script  

  * cloud-init (ubuntu 20.04 LTS, 8 GB Ram) 

```
#!/bin/bash 

groupadd sshadmin
USERS="11trainingdo"
echo $USERS
for USER in $USERS
do
  echo "Adding user $USER"
  useradd -s /bin/bash --create-home $USER
  usermod -aG sshadmin $USER
  echo "$USER:deinsehrgeheimespasswort" | chpasswd
done

# We can sudo with 11trainingdo
usermod -aG sudo 11trainingdo 

# 20.04 and 22.04 this will be in the subfolder
if [ -f /etc/ssh/sshd_config.d/50-cloud-init.conf ]
then
  sed -i "s/PasswordAuthentication no/PasswordAuthentication yes/g" /etc/ssh/sshd_config.d/50-cloud-init.conf
fi

## both is needed 
sed -i "s/PasswordAuthentication no/PasswordAuthentication yes/g" /etc/ssh/sshd_config

usermod -aG sshadmin root

# TBD - Delete AllowUsers Entries with sed 
# otherwice we cannot login by group 

echo "AllowGroups sshadmin" >> /etc/ssh/sshd_config 
systemctl reload sshd 

echo "Installing microk8s"
snap install --classic --channel=1.23/stable microk8s
microk8s enable dns rbac
echo "alias kubectl='microk8s kubectl'" >> /root/.bashrc
source ~/.bashrc
alias kubectl='microk8s kubectl'

# now we need to modify the setting of kube-api-server
# currently in 1.23 no other admission-plugins are activated
echo "--enable-admission-plugins=PodSecurityPolicy" >> /var/snap/microk8s/current/args/kube-apiserver
microk8s stop
microk8s start


```

## Step 2: 

```
# Setup .kube/config from content
microk8s config
```

## Step 3

```
# rbac.yaml 
# vi service-account.yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: training
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pods-clusterrole
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list", "create"]
- apiGroups: [""] # "" indicates the core API group
  resources: ["events"]
  verbs: ["get", "list", "create"]
---
# vi rb-training-ns-default-pods.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: rolebinding-ns-default-pods
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: pods-clusterrole
subjects:
- kind: ServiceAccount
  name: training
  namespace: default
```

## Step 4: Secret aus secrets rauskopiert

```
kubectl get secrets | grep trainingtoken 
kubectl get secrets training-token<xyz> -o yaml 
# reinkopiert
TOKEN=$(echo fsafkdsafkafksadfksafkdsaa | base64 -d) 
```
```
echo $TOKEN
kubectl config set-context training-ctx --cluster microk8s-cluster --user training
kubectl config set-credentials training --token=$TOKEN

```

## Step 5: Apply yaml-manifests for psp - stuff (as admin)

```
# vi setup.yaml 
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: norootcontainers
spec:
  allowPrivilegeEscalation: false
  allowedHostPaths:
  - pathPrefix: /dev/null
    readOnly: true
  fsGroup:
    rule: RunAsAny
  hostPorts:
  - max: 65535
    min: 0
  runAsUser:
    rule: MustRunAsNonRoot
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  volumes:
  - '*'
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: norootcontainers-psp-role
rules:
- apiGroups:
  - policy
  resourceNames:
  - norootcontainers
  resources:
  - podsecuritypolicies
  verbs:
  - use
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: norootcontainers-psp-role:training
  namespace: default
roleRef:
  kind: ClusterRole
  name: norootcontainers-psp-role
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: training
  namespace: default
```  
 
## Step 5: Change to training-ctx and apply 

```
kubectl config use-context training-ctx 
```

```
# vi demopod.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: demopod
spec:
  containers:
    - name:  demopod
      image: nginx
```

```
kubectl apply -f demopod.yaml
kubectl get pods ## expecting
kubectl describe pods demopod 
```





## Reference 

  * https://docs.mirantis.com/mke/3.4/ops/deploy-apps-k8s/pod-security-policies/psp-examples.html
