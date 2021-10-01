* create pod
```
kubectl run redis --image=redis:alpine -l "tier=db" [--port=5701] [--expose]
note: --expose actually creates a svc to expose the port

--command -- <cmd> <arg1> ... <argN> [extra command to be used in pod ]
```

* svc:
```a
kubectl create svc clusterip redis --tcp=6379:637
```

* deploy 
```aidl
kubectl create deploy webapp --image=kodekloud/webapp-color -r 3
```



* configmap:
`kubectl create configmap webapp-config-map --from-literal=APP_COLOR=darkblue`
  
use configmap:
```
- env:
    - name: APP_COLOR
      valueFrom:
        configMapKeyRef:
          name: webapp-config-map
          key: APP_COLOR
```

* secrets:
`kubectl create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123`
  
using secret:
```
envFrom:
      - secretRef:
          name: mysecret
```