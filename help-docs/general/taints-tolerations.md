```
tolerations:
 - key: "app"
   operator: "Equal"
   value: "blue"
   effect: "NoSchedule"
```

remove taint:
`kubectl taint nodes controlplane node-role.kubernetes.io/master:NoSchedule-`