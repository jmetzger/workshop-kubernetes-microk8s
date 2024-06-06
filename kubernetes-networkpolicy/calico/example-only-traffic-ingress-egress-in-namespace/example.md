# Example only allow traffic within Namespace and Egress 

```
---
apiVersion: crd.projectcalico.org/v1
kind: GlobalNetworkPolicy
metadata:
  name: default-deny
spec:
  namespaceSelector: kubernetes.io/metadata.name != "kube-system"
  types:
  - Ingress
  - Egress
  egress:
   # allow all namespaces to communicate to DNS pods
  - action: Allow
    protocol: UDP
    destination:
      selector: 'k8s-app == "kube-dns"'
      ports:
      - 53
  - action: Allow
    protocol: TCP
    destination:
      selector: 'k8s-app == "kube-dns"'
      ports:
      - 53
---
# 03-allow-ingress-my-nginx.yml
apiVersion: crd.projectcalico.org/v1
kind: NetworkPolicy
metadata:
  name: allow-only-from-within-namespace
  namespace: fromhere
spec:
  types:
  - Ingress
  - Egress
  egress:
  - action: Allow
  ingress:
  - action: Allow
---
# 03-allow-ingress-my-nginx.yml
apiVersion: crd.projectcalico.org/v1
kind: NetworkPolicy
metadata:
  name: allow-only-from-within-namespace
  namespace: np
spec:
  types:
  - Ingress
  - Egress
  egress:
  - action: Allow
  ingress:
  - action: Allow
    source:
      namespaceSelector: kubernetes.io/metadata.name == "np"
```
