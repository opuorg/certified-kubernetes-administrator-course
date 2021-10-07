#### Instruction doc: 
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

1. Make sure linux nodes are ready
    - Have network connectivity between each other
2. ssh into each node. 
3. Letting iptables see bridged traffic
4. Installing runtime
   - Docker: https://kubernetes.io/docs/setup/production-environment/container-runtimes/#docker
   Note: Docker documentation doesn't work sometimes for adding the gpg key and repository. In that case, this url can be used. https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04
5. Installing kubeadm, kubelet and kubectl 
6. Creating a cluster with kubeadm
   - Make sure to specify a CIDR for pod network that doesn't conflict with node cidr. This need to be passed as `--pod-network-cidr`.
   - Specify the api server address to pass as `--apiserver-advertise-address=<ip-address>` argument. In local cluster case, it is normally ip of master node.
7. Init kubeadm in master (This is only needed in master node)
   `kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.56.2`
   Important note: Copy the kubeadm join command from output. 
8. Setup networking using this command
`kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"`
9. Setup nodes:
   - ssh into node
   - be sudo
   - execute the copied command.
   


## Notes:
- see versions: `sudo apt-cache madison kubeadm`
