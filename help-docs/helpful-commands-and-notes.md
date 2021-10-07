ETCD – Commands:
```
etcdctl snapshot save
etcdctl endpoint health
etcdctl get
etcdctl put
```

To set the right version of API set the environment variable ETCDCTL_API command
- When API version is not set, it is assumed to be set to version 2. And version 3 commands listed above don’t work. When API version is set to version 3, version 2 commands listed above don’t work.

Apart from that, you must also specify path to certificate files so that ETCDCTL can authenticate to the ETCD API Server. The certificate files are available in the etcd-master at the following path. We discuss more about certificates in the security section of this course. So don’t worry if this looks complex:
```
–cacert /etc/kubernetes/pki/etcd/ca.crt
–cert /etc/kubernetes/pki/etcd/server.crt
–key /etc/kubernetes/pki/etcd/server.key
```

```
kubectl run nginx --image nginx
```

## Notes:
- each node has kubelet as service
- each node has kube-proxy which is a daemonset
    - it uses it's config file from a configmap. 