# Setup macvlan 

## Restrictions 

 * It is not possible to ping pods with that macvlan - ip, if it is on another host
 * This is because every ip gets its on mac
 * You can circumvent this by setting promisc mode (not wanted in most cases)

## Set promisc mode on/off for a specific interface 

```
ip link set eth1 promisc on
# you will be able to set in the interface line
ip a | grep -i promisc
# 3: eth1: <BROADCAST,MULTICAST,PROMISC,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
ip link set eth1 promisc off
```


```
