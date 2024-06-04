# Setup iscsi 

## Step 1: Install dependencies / client libraries (ubuntu) 

```
sudo apt install open-iscsi sudo modprobe iscsi_tcp
```

## Step 2: Create persistent volume 

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: iscsi-pv
spec:
  capacity:
    storage: 12Gi
  accessModes:
    - ReadWriteOnce
  iscsi:
     targetPortal: 192.0.2.100:3260
     iqn: iqn.2017-10.local.example.server:disk1
     lun: 0
     fsType: 'ext4'
     readOnly: false
```

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: iscsi-pvc
  namespace: default
spec:
  storageClassName: ""
  volumeName: iscsi-pv
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 8Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: iscsi-test
spec:
  containers:
    - name: www
      image: nginx
      volumeMounts:
        - name: html
          mountPath: "/var/www/html"
  volumes:
    - name: html
      persistentVolumeClaim:
        claimName: iscsi-pvc
```

## Reference 

  * Reference: https://docs.mirantis.com/mke/3.4/ops/deploy-apps-k8s/persistent-storage/configure-iscsi.html
