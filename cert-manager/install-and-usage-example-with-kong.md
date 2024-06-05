# Install Cert-Manager and usage 

## Step 1: Install Cert Manager 

```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.5/cert-manager.crds.yaml
## Add the Jetstack Helm repository
helm repo add jetstack https://charts.jetstack.io

## Install the cert-manager helm chart
helm install cert-manager --namespace cert-manager --create-namespace --version v1.14.5 jetstack/cert-manager

```

## Step 2: Create token for digitalocean 

```
# You can create your token here
https://cloud.digitalocean.com/account/api/tokens/new

# now you need to encode it
echo -n 'your-access-token' | base64
```

## Step 2.5: Subdomains einrichten in digitalocean - dns 

```
TBD 
```

## Step 3: Create secret based on that 

```
cd
mkdir -p manifests
cd manifests
mkdir ssl
cd ssl
```

```
vi 01-secret.yml
```

```
apiVersion: v1
kind: Secret
metadata:
  name: digitalocean-dns
data:
  # insert your DO access token here
  access-token: "base64 encoded access-token here"
```

```
kubectl -n cert-manager apply -f .
```

## Step 4: Create an clusterissuer (works across all namespaces) 

```
vi 02-clusterissuer.yml
```

```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod-issuer
spec:
  acme:
    email: some@domain.de
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - selector:
        dnsZones:
          - "do.t3isp.de"
      dns01:
        digitalocean:
          tokenSecretRef:
            name: digitalocean-dns
            key: access-token
```

```
kubectl apply -f .
```

## Step 5: manifests for certificate (wildcard) 

```
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: le-crt
spec:
  secretName: tls-secret
  issuerRef:
    kind: ClusterIssuer
    name: letsencrypt-prod-issuer
  commonName: "*.app1.do.t3isp.de"
  dnsNames:
    - "*.app1.do.t3isp.de"
```

```
kubectl apply -f .
```

## Step 6: check if certificate was created 

```
kubectl get certificates
kubectl describe certificates let-cert 
kubectl get secret tls-secret -o yaml 
```

## Step 7: Setup gateway with https 

```
vi 04-gateway.yml
```

```
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: prod-https-kong
spec:
  gatewayClassName: kong
  listeners:
  - protocol: HTTPS
    port: 443
    name: prod-https-gw
    hostname: "jochen.app1.do.t3isp.de"
    tls:
      certificateRefs:
      - kind: Secret
        name: tls-secret
    allowedRoutes:
      namespaces:
        from: Same
```

```
kubectl apply -f .
```

```
vi 05-http-routes.yml
```

```
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
 name: apple-https-route
spec:
  parentRefs:
  - name: prod-https-kong
  hostnames:
  - jochen.app1.do.t3isp.de
  rules:
  - backendRefs:
    - name: apple-service-kong
      port: 80
```

```
kubectl apply -f .
```

### Step 8: Test in browser

```
https://jochen.app1.t3isp.de
```


## Reference 
  * https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nginx-ingress-with-cert-manager-on-digitalocean-kubernetes
  * Problem fix: https://www.digitalocean.com/community/questions/how-do-i-correct-a-connection-timed-out-error-during-http-01-challenge-propagation-with-cert-manager
  * Question: Do we have the same problem wiht metallb
  * https://cert-manager.io/docs/configuration/acme/dns01/digitalocean/
  * https://artifacthub.io/packages/helm/cert-manager/cert-manager
