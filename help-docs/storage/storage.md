## 
csi: Container storage interface

####  volume to host directly (hostPath):
```
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: event-simulator
    image: kodekloud/event-simulator
    env:
    - name: LOG_HANDLERS
      value: file
    volumeMounts:
    - mountPath: /log
      name: log-volume

  volumes:
  - name: log-volume
    hostPath:
      # directory location on host
      path: /var/log/webapp
      # this field is optional
      type: Directory
```

* persistent volumes: clusterwise pool of volumes 
  
Configure pv using `hostpath`:
```
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv-vol1
spec:
  accessModes: [ "ReadWriteOnce" ]
  persistentVolumeReclaimPolicy: Recycle
  capacity:
    storage: 1Gi
  hostPath:
    path: /tmp/data
```

* persistent volume claims: claim a volume from pool of pvs. Admin creates pv, user claims pvc
pvc defininition:
```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
    name: myclaim
spec:
    accessModes: [ "ReadWriteOnce" ]
    resources:
      requests:
        storage: 1Gi
```

* accessModes: 

* using in pods:

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
  - name: mypd
    persistentVolumeClaim:
      claimName: myclaim
```


# Storage Class:
Create claim using storageClass:
```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
    name: local-pvc
spec:
    accessModes: [ "ReadWriteOnce" ]
    resources:
      requests:
        storage: 500Mi
    storageClassName: local-storage
```

StorageClass creation:
```
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: delayed-volume-sc
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```