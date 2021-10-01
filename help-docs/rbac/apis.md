* api groups are what comes after url base domain, such as /version, /api, /apis etc.

* there are two categories of apis:
- api:
    - core: core func
        - ns, pods, rs, events, nodes, pv, pvc, svc etc.
- apis: 
    - named: new version
        - apps, extensions, networking.k8s.io, storage.k8s.io, authentication.k8s.io, certificates.k8s.io
            - apps:
                - v1:
                    - deployments
                    - replicasets
                    - statefulset
        - networking.k8s.io:
            - v1
                - networkpolicies
                - 
        - certificates.k8s.io:
            - v1
                - 

* named group is the new norm

** List all api groups
`kubectl api-resources -o wide`

# api can be accessed using browser or curl using kube proxy command
- `kubect proxy`
