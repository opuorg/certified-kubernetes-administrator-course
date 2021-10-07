- Access service:
  - one method could be that use the ip of the svc and the port
  - another is this way:
```
web-service.default.svc.cluster.local has address 10.106.112.101

web-service.default.svc.cluster.local has address 10.106.112.101

web-service.default.svc.cluster.local has address 10.106.112.101

web-service.default.svc.cluster.local has address 10.106.112.101
```


- access service between ns:
  * same ns: just service name
  * other ns template:
    `<svc>.<ns>.svc.cluster.local`
  

- Create service:
```
kubectl create svc nodeport NAME [--tcp=port:targetPort] [--dry-run=server|client|none]
```

```
---
apiVersion: v1
kind: Service
metadata:
  name:
spec:
  type:
  ports:
    - targetPort:
      port:
      nodePort:
  selector:
    name:
```