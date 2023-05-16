# Pod Security Policy (PSP)  

## General 

  * PodSecurity is an eine Rolle gebunden (clusterrole)
  * Deprecated in 1.21 removed in 1.25
  * From 1.25 on please use PSA (Pod Security Admission) instead 

## Prerequisites 

  * We should have a running Cluster of 1.22/1.23


## Walkthrough 

### Digitalocean microk8s 1-node - cluster 

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



## Walkthrough using kind (Windows) 

### Step 1: Download Kind and install (Windows WSL) 

```
# if not yet (as admin)
# wsl --install 

curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.18.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```




## Walkthrough (allowPrivilegeEscalation: false)

### Step 1: Create PodSecurityPolicy (Kubernetes V1.22. policy/v1beta1 

```
cat <<EOF | kubectl apply -f -
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
EOF
```

## Step 2: Set the role

```
cat <<EOF | kubectl apply -f -
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
EOF


```

## Step 3: Set the rolebinding 

```



```


## Step 2: Create configuration for cluster 

```
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
kubectl apply -f .
kubectl describe po demopod
```

```
### maybe we use something else --> above 

# We found that role through 
# kubectl get clusterrole cluster-admin -o yaml 
# kubectl get clusterrolebinding cluster-admin -o yaml
# -> system:masters 

#### This will only work, if you are connected by certificate with a user that 
#### is in the group system:masters 
#### You will be able to see that in the client certificate
# echo "CERTIFICATE DATA FROM KUBECONFIG" | base64 -d > test.crt 
# openssl x509 -in test.crt -text -noout

cat <<EOF | kubectl apply -f -
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: norootcontainers-psp-role:group:system:masters
roleRef:
  kind: ClusterRole
  name: norootcontainers-psp-role
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: Group
  name: system:masters 
EOF

```

## Step 3: Test to create a pod

```


```


## Reference 

  * https://docs.mirantis.com/mke/3.4/ops/deploy-apps-k8s/pod-security-policies/psp-examples.html
