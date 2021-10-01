* There are different authorization mechanisms supported by kubernetes
    - Node Authorization (per node e.g kubelet)
    - Attribute-based Authorization (ABAC): pass permissions as attributes during creation of user
    - Role-Based Authorization (RBAC): create roles with permissions (standard)
    - Webhook: using a different software for auth, use webhook. 
    - always allow:
    - always deny:
    
- Auth mode is set in kubeapiserver process as argument --authorization-mode=AlwaysAllow
    - default "AlwaysAllow"
    
- multiple modes can be used using comma "Node,RBAC,Webhook"
    - its processed in order it was set in the argument. like 1. node, 2. rbac, 3. webhook


# RBAC:
1. Create a role object with all the permissions
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "list", "update", "delete", "create"]
- apiGroups: [""]
  resources: ["ConfigMap"]
  verbs: ["create"]
```
- For core groups, "" should be ok. for any other groups, need to specify groups. 
- resources: all resources that need permission to
- verbs: what can the do on the resouce. 

2. RoleBinding: It links user to role. 
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devuser-developer-binding
subjects:
- kind: User
  name: dev-user # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

#### Roles and RoleBinding are per ns based, if none specified, it is considered default ns, but if different ns needed, it needs to be added in metadtaa section.

- Check access:
`k auth can-i create pod`
  
- check access as another user by admin:
`k auth can-i create pod -as user1`
  
Note: `resourceNames` field can be added to role to restrict to a specific resource.
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "list", "update", "delete", "create"]
  resourceNames: ["blue","yellow"]
```

** List all api groups
`kubectl api-resources -o wide`

- See list of api groups resources that are namespace scoped, use command
  `kubectl api-resources --namespaced=true`
and non namesapced
`kubectl api-resources --namespaced=false`

# ClusterRoles:
- some resources are cluster roles, to create roles for those, we need cluster roles. 
- Clusterrole: same as role, but for non ns
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["nodes"]
  verbs: ["get", "list", "delete", "create"]
```

- ClusterRoleBinding: 
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-role-binding
subjects:
- kind: User
  name: cluster-admin # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```


# ServiceAccounts:
1. create sa:
k create sa acc-name
2. it uses a token, which automatically created. 
- a secret is created to take that token. 
- that token can be used to authenticate to the cluster. 
3. Access role can be granted to the SA to grant access to specific resources. 
4. that token can be mounuted to applications to automatcially grant them access. 

* a field "serviceAccountName:" can be added with a different service account name as value to mount that SA in the pod.
* another field "autoMountServiceAccount:" can be set to false to disable mounting the token.

