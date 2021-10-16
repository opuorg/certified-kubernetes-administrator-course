- static pod: 
- There are few pods in kuberenetes that are not part of a deploy/ds etc. They are placed in a special directory in each node.
    - any pod defininition placed in here automatically spin up, even after deleting the pod. 
    - each node has this. 
    - kubelet handles this. 
    - directory is - /etc/kubernetes/manifests/ 
    - This location is passed as arg `--pod-manifest-path` in kubelet process
    - This location could be also added as an option to in a config file which is added in kubelet process as arg `--config`
    - on that config file, the option is added like this `--staticPodPath:`
    