## Cluster Upgrade:
 - different versions strategy:
    - nothing should be higher version then kube apiserver
    - controller manager and kube scheduler can be one version lower
    - kubelet and kube proxy can be two versions lower. 
    - kubectl can be one version higher, same or one version lower. 
      
        (Note: Version mentioned above is .x)
    
- Kubernetes only support 3 back versions
- upgrade one major version at a time


kubeadm upgrade process:
1. `kubeadm upgrade plan`: show all the info about versions, available, including all controlplane component, their versions etc.
it also shows that kubelet version need to be manually upgraded.
2. upgrade kubeadm in master node
   - if needed, Determine which version to upgrade to
      ```
      apt update
      apt-cache madison kubeadm
      ```
   - perform update
      `apt-get upgrade kubeadm=1.12.0-00`
   - verify version
      `kubeadm version`
   - verify plan again
      `kubeadm upgrade plan`
     * Note: doesn't matter what the plan says, always upgrade to same version as kubeadm which we upgraded above. 
     
3. Perform upgrade:
`kubeadm upgrade apply v1.12.0`
   - if we perform kubect get node now, it will still show current version, cz kubelet is not upgraded yet. 
4. upgrade kubelet:
   `apt-get upgrade kubelet=1.12.0-00`
   * kubelet might not be in master node depending on how the cluster is setup. kubeadm has it, but from scratch don't.

4.1. upconrdon node
     
5. Now upgrade worker nodes.
   - drain node.
   - upgrade kubeadm
     `apt-get update`
     `apt-get upgrade kubeadm=1.12.0-00`
   - upgrade kubelet
     `apt-get upgrade kubelet=1.12.0-00`
   - upgrade node config:
      `kubeadm upgrade node config --kubelet-version v1.12.0`
  - restart kubelet service
      `systemctl restart kubelet`
  - un cordon node.
6. Repeat for each node. 

Notes:
- for versions/help, use kubernetes kubeadm upgrade docs
- katacoda can be used for practicing
- documentation available in kubernetes documentation page, search for `kubeadm upgrade`
