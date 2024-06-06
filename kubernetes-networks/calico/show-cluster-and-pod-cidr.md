# Important resource commands (works on new version with calico-api-server) 

## ippools 

```
# show the cluster-cidr used 
kubectl get ippools -o yaml 
```

## pod-cdir 

```
kubectl get ipamblocks -o yaml | less
kubectl get ipamblicks -o yaml > myblocks  
```
