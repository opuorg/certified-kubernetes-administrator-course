* manual scheduling:
    - either via setting nodeName filed during creating
        - nodeName: field is same level as containers, under spec
    - or use api and nodeBinding object
    
- multiple scheduler
- can copy /etc/kubernetes/manifest/kube-scheduler.yaml file
- need to change following params
```
- --leader-elect=false
- --port=10282
- --scheduler-name=my-scheduler
- --secure-port=0  
```

- schedule using custom scheduler
