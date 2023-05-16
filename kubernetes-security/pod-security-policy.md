# Pod Security Policy (PSP)  

## General 

  * PodSecurity is an eine Rolle gebunden (clusterrole)
  * Deprecated in 1.21 removed in 1.25
  * From 1.25 on please use PSA (Pod Security Admission) instead 

## Prerequisites 

  * We should have a running Cluster of 1.22 
  * In our case, we have started that with Rancher Desktop and used that version 

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
