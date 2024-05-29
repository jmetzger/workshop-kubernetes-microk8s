# Übung http-route mit nginx gateway 

## Schritt 1: Gateway Api Ressources installieren 

```
# Standard - Stabile Features 
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.0.0/standard-install.yaml

# Es gibt auch einen experimentellen Channel, der aber anders installiert werden muss
# MACHEN WIR NICHT 
# kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.0.0/experimental-install.yaml
```

## Schritt 2: nginx fabric gateway mit helm installieren 

```
helm install ngf oci://ghcr.io/nginxinc/charts/nginx-gateway-fabric --create-namespace -n nginx-gateway
```

## Reference: 

  * https://github.com/nginxinc/nginx-gateway-fabric
  * https://docs.nginx.com/nginx-gateway-fabric/installation/expose-nginx-gateway-fabric/
