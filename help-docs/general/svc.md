#### service Types:
  - nodeport
  - clusterip
  - load balander

#### Nodeport:
- this type of services listens to a port in node and forward traffic to a pod
- Three ports involved:
  - TargetPort: actual webserver port from pod
  - port: port for the service itself
  - noePort: The port in node that will be used to access the service from internet.
- Valid nodeport range: 30000-32767
  - when nodeport not defined, a port will be automatically added.
- port is array, so multiple ports can be added.

#### ClusterIp:
- This type creates a virtual ip inside the cluster to enable communicatino between services
- When type is not specified, its by default clusterIP
  - port: port of the service
  - targetPort: port of pod
- selector is used to point to pod

#### LoadBalancer:
- cloud load balancer used as service. 

#### load balancers:



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