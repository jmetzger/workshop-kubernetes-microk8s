# FAQs

## Welcher DNS wird genommen (microk8s)

  * Microk8s verwendet standardmäßig die Nameserver, die auf dem Host-System /etc/resolv.conf eingetragen

```
By default it forwards requests to the system-defined servers in /etc/resolv.conf for resolving
addresses. This can be changed when you enable the addon, for example:
```

  * https://microk8s.io/docs/addon-dns

## Fehler: Nameserver limits were exceeded, some nameservers have been omitted, the applied nameserver line is: 67.207.67.3 67.207.67.2 67.207.67.3

  * Fehler tritt in: calico-node - pods auf kube-system auf. 
  * https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/#known-issues
  * Fix ? 
