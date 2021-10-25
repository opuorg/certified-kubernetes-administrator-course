```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      name: internal
  policyTypes:
  - Egress
  - Ingress
  ingress:
    - {}
  egress:
  - to:
    - podSelector:
        matchLabels:
          name: mysql
    ports:
    - protocol: TCP
      port: 3306

  - to:
    - podSelector:
        matchLabels:
          name: payroll
    ports:
    - protocol: TCP
      port: 8080

  - ports:
    - port: 53
      protocol: UDP
    - port: 53
      protocol: TCP

```

- spec.podSelector: which pod to apply the policy on, uses label to determine
```
ingress:
- from:
  - podSelector: 
        matchLabels:
            name: label
    namespaceSelector: 
        name: ns
  - ipBlock:
        cidr: xx.xx.xx.xx/xx
        
  ports:
  - protocol:
    port: 
```