- nodes has to be 64 bit OS
- In big clusters, ctcd cluster can be placed in a different node

## HA Basics:
- When master node not available:
    - k can't access 
    - pods can't restart/scale up etc.
    
- Where to point kubect:
    - We configure a load balancer for api server. in AWS case, ELB.
    
Other things:
- controller manager: They work as active/stand by mode.
    - it is defined in --leader-elect options. 
    - it is true by default. 
    - whoever processes the first rquest become leader
    - 
    
## External etcd server:
- put etcd server in a different node. 
- kube-apiserver process/pod has arg --etcd-servers=https://url1:port,https://url2:port defining where to find etcd server.
- we can specify a list of etcd servers. 

## HA etcd:
- etcd is distributed
- for etcd with multiple pods:
    - There will be a leader and workers
    - when reading data, client can read from any 
    - when writing
        - if comes to leader, leader process the request, and make sure all workers get a copy. 
        - if comes to a worker, it doesn't process, but forward to master.
    - Leader election process:
        - Initially, it uses RAFT protocol, and whoever completes the process first, notifies other then he's trying to become a leader.
        - After that, leader continues notifying others every 2 secons, that it is still leader. 
        - if a worker doesn't hear from leader for some time, it tries to be lader
    
- A write is considered complete when it is copied to at least 2 nodes (including master) [ in a 3 node cluster]
    - it follows `Majority = n/2 + 1`
    
