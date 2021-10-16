scheduling:
- kube-scheduler handles scheduling. 
    - it looks for any pod that don't have 'nodeName' field not set. when found, it adds that label to pod based on node availability and pod requirement and put it in node.

* manual scheduling:
    - either via setting nodeName filed during creating
        - nodeName: field is same level as containers, under spec
    - or use api and nodeBinding object
    
- multiple scheduler:
  - when creating multiple scheduler, other schedulers should have unique name. 
  - Also, --leader-elect need to be false custom-scheduler when only one master node.
    - when multiple master and custom scheduler, keep `--leader-elect=true`, but also add `--lock-object-name=<custom-name>`
- the custom scheduler pod definition can be placed in `/etc/kubernetes/manifest/`
- the definition file can be copied from existing `kube-scheduler.yaml` file

- to have a pod use customer scheduler, `schedulerName:` field will need to be added in same label as `container`
  
- need to change following params
```
- --leader-elect=false
- --port=10282
- --scheduler-name=my-scheduler
- --secure-port=0  
```
