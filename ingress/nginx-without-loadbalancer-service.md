# Install ingress-nginx without Service - LoadBalacner 

## Why ?

  * We want to use service on our own (own manifest) for specific settings metallb (ippool a.s.o)

## Walkthrough 

```
cd
mkdir -p manifests
cd manifests
mkdir ingress
cd ingress
vi values.yml
```

```
controller:
  service:
    enabled: false
    external:
      enabled: true
```

```
helm install nginx-ingress ingress-nginx/ingress-nginx -f values.yml --namespace ingress helm install nginx-ingress ingress-nginx/ingress-nginx --namespace ingress
```
