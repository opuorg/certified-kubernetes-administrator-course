## CSI
csi: Container storage interface

## Volume Types:
- hostpath:
```
volumes:
- name: local-volumen
  hostPath:
    path: <path-in-host>
    type: Directory
```
- EBS (AWS):
```
volumes:
- name: aws-volumen
  awsElasticBlockStore:
    volumeID: <ebs-id>
    fsType: ext4
    
```

- Example of hostpath in use:
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

## persistent volumes: 
- cluster wise pool of volumes that can be used by pods as needed.
- template (hostpath):
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```
- template (aws ebs):
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  awsElasticBlockStore:
    volumeId:
    fsType: ext4
```

## AccessModes:

## reclaimPolicy:

## Persistent volume claims:
- Admin creates pvs to be used. 
- Users create pvc to use in pods from the pvs
- template:
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```

# Storage Class:
- when creating a cloudstorage pv, the volume has to be present already. 
- in prod env, this is hideous task. We need auto provision in prod. That is where storage class comes in. 
- This is called dynamic provision. 
- For that we create a storage class definition object. 
StorageClass creation examples:
  
- no provisioner (local):
```
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: delayed-volume-sc
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```
- aws ebs:
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Retain
allowVolumeExpansion: true
mountOptions:
  - debug
volumeBindingMode: Immediate
```
- in storage class case, we don't need to create pv. its automatically created.
- different provisioners have different parameters that need to be defined in storage class definition. 
    - local doesn't.


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