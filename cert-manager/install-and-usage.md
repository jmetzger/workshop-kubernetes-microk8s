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

## Step 4: Create an issuer 

```
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: example-issuer
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




## Reference 
  * https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nginx-ingress-with-cert-manager-on-digitalocean-kubernetes
  * Problem fix: https://www.digitalocean.com/community/questions/how-do-i-correct-a-connection-timed-out-error-during-http-01-challenge-propagation-with-cert-manager
  * Question: Do we have the same problem wiht metallb
  * https://cert-manager.io/docs/configuration/acme/dns01/digitalocean/
