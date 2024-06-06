# Important resource commands (works on new version with calico-api-server) 

## ippools 

```
# show the cluster-cidr used 
kubectl ippools -o yaml 
```

## pod-cdir 

```
kubectl ipamblocks -o yaml | less
kubectl ipamblicks -o yaml > myblocks  
```
