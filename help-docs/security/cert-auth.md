## Cert info
* apiserver uses apiserver.crt and apiserver.key for authenticating requests.
* etcserver users etcdserver.crt and etcserver.key
* kubelet.crt and kubelet.key used by kubelet
  - [ it can be different based on how or who setup]
  
* Clients (users and other kube components) uses cert to communicate with these servers. 
    - users might use admin.crt and admin.key to connect apiserver via kubect 
    - scheduler might use scheduler.crt and scheduler.key to connect to apiserver
    - controller manager use another set of certs to connect to kube apiserver
    - kube proxy use another set of certs to connect to kube apiserver
    
* servers also connects to each other, one acts as client, another as server. 
    - kubeapi server connects to etcd server 
        - they can use the same certs as client to connect to other server.
        - or a different set of certs can be generated and used for that purpose. 
    - kube api server also connects to kubelet server
    
## Generate Certs
* To generate all the certs, we need minimum a CA which kubernetes will trust. 
    - there can be multiple CA, example, one ca for all and another for etcd only
- kubeadm deploys all the certs automatically
    - kubeadm generated cert lists: https://github.com/mmumshad/kubernetes-the-hard-way/blob/master/tools/kubernetes-certs-checker.xlsx

## Check Certs
- when kubeadm generated env, look at /etc/kubernetes/manifests/kube-apiservers.yaml file for all the cert kubeapiserver uses. 
- to see info on cert use openssl command
`openssl x509 -in /path/to/cert -text -noout`
  

## Things to troubleshoot:
- Issuer, Domain, expire date, organization, 
- standard: https://kubernetes.io/docs/setup/best-practices/certificates/#certificate-paths
- check server logs
  $ journalctl -u etcd.service -l
  
- when setup using kubeadm, check etcd-master pod logs
- also check docker logs 
