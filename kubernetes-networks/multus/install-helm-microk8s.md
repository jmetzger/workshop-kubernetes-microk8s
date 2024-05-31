# Install multus on microk8s 

## No microk8s enable multus please ! 

  * Only available in the community repo
  * For production we do not want to use community stuff

## Better: Install by helm (use bitnami) 

  * There are different sources, but we use bitnami
  * .. because we trust them
  * https://artifacthub.io/packages/helm/bitnami/multus-cni

## Install - modification 

  * Example in artifacthub.io install it in default,
  * but we want that in kube-system
  * We always install a specific version 

```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install multus-cni bitnami/multus-cni --namespace kube-system --version 2.1.0
```

## Reference:

  * https://github.com/k8snetworkplumbingwg/multus-cni/tree/master
