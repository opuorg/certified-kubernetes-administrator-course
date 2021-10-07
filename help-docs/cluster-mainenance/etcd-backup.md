## documentations for backups 
https://discuss.kubernetes.io/t/etcd-backup-and-restore-management/11019

# etcd
- etcd is a service 
- --data-dir is flag who sets where all etcd files are stored. 
- etcdctl is a bin available to perform tasks like snapshot, restore etc.
    - etcdctl snapshot save <etcd.db>
    
To backup:
`etcdctl snapshot save <etcd.db>`

To resotre:
1. Stop kube-api server
    `service kube-apiserver stop`
   
2. restore
    `etcdctl snapshot restore <etcd.db> --data-dir </path/to/new-dir`
2.1. update the /etc/kubernetes/manifests/etcd.yaml with new path. we are just updating the volume mount path, so the directory structure inside the path stays the same.
```
  volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd1
      type: DirectoryOrCreate
    name: etcd-data
```
   
3. reload the  daemon-reload
    `systemctl daemon-reload`
4. restart
    `service etcd restart`
5. start api service
   `service kube-apiserver start`
Notes: 
   - with all the ectdctl command, we have to specify these options
```
--endpoints=https://127.0.0.1:2379 \
--cacert=/etc/etcd/ca.crt \
--cert=/etc/etcd/etcd-server.crt \
--key=/etc/etcd/etcd
```

- set ETCDCTL_API to 3 before each command or do this for sesssion `export ETCDCTL_API=3`
- -h is also available


Tips:
- when taking snapshot, kubernetes docs can be used. 
- same with restore, but make sure to add `--data-dir=<new-dr>` in restore command, 
  - then edit volume location for etcd pod in static pod directory to point to new dir.
