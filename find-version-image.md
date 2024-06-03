# Image Version einer Software rausfinden 

```
kubectl -n metallb-system describe pods metallb-controller-665d96757f-whwrm
kubectl -n metallb-system describe pods metallb-controller-665d96757f-whwrm | grep -i "image:"
```

```
kubectl -n metallb-system get deploy metallb-controller -o yaml
```
