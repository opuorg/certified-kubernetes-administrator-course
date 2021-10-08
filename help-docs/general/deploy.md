- update a deploy and record the change:
`kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1 --record`
- to expose pod to dns, this needs to be added to container
```
ports:
- name: http
  containerPort: 80
```

- pod dns:
```
pod-ip-address.my-namespace.pod.cluster-domain.example
```