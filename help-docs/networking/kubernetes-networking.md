# Cluster networking:
- Each node should have unique mac and ip (when cloning, be careful)
- all required port needs to be opened. 
#### References:
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports
- https://kubernetes.io/docs/concepts/cluster-administration/networking/

At this moment in time, there is still one place within the documentation where you can find the exact command to deploy weave network addon:
search `Apply the CNI plugin of your choice`
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/#steps-for-the-first-control-plane-node (step 2)


## practice:
- find out net int used for node to node comm
    - get ip of mster node
    - use command `ip a` and search for the int with that ip
- `ip link show eth0`: see info of interface
- see ip and hardware assigned to a node, try this command from master
    `arp node01`
- show default gateway: `ip route show default`
- see list of ports and process: `netstat -nplt`

# Pod networking:
- NEED TO STUDY AGAIN

# CNI:
- CNI plugin is configured in the kubelet service, provided as argument
  - it also has `cni-bin-dir`, `cni-conf-dir` etc as arguments
  - `/opt/cni/bin` is default `cni-bin-dir`
  - `/etc/cni/net.d/` is default `cni-conf-dir`
    
- in conf directory, there's a file called `bridge.conf` which defined which cni binary to use.
    - if there's multiple conf files, they are used in alphabetical order

#### Weave Cni:
- Check if have time 
- Deployment:
`kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"`
  
- add env value like this `&env.WEAVE_MTU=1337` after url, inside quote.
List of options to set:
  
```
The list of variables you can set is:

CHECKPOINT_DISABLE - if set to 1, disable checking for new Weave Net versions (default is blank, i.e. check is enabled)
CONN_LIMIT - soft limit on the number of connections between peers. Defaults to 200.
HAIRPIN_MODE - Weave Net defaults to enabling hairpin on the bridge side of the veth pair for containers attached. If you need to disable hairpin, e.g. your kernel is one of those that can panic if hairpin is enabled, then you can disable it by setting HAIRPIN_MODE=false.
IPALLOC_RANGE - the range of IP addresses used by Weave Net and the subnet they are placed in (CIDR format; default 10.32.0.0/12)
EXPECT_NPC - set to 0 to disable Network Policy Controller (default is on)
KUBE_PEERS - list of addresses of peers in the Kubernetes cluster (default is to fetch the list from the api-server)
IPALLOC_INIT - set the initialization mode of the IP Address Manager (defaults to consensus amongst the KUBE_PEERS)
WEAVE_EXPOSE_IP - set the IP address used as a gateway from the Weave network to the host network - this is useful if you are configuring the addon as a static pod.
WEAVE_METRICS_ADDR - address and port that the Weave Net daemon will serve Prometheus-style metrics on (defaults to 0.0.0.0:6782)
WEAVE_STATUS_ADDR - address and port that the Weave Net daemon will serve status requests on (defaults to disabled)
WEAVE_MTU - Weave Net defaults to 1376 bytes, but you can set a smaller size if your underlying network has a tighter limit, or set a larger size for better performance if your network supports jumbo frames - see here for more details.
NO_MASQ_LOCAL - set to 0 to disable preserving the client source IP address when accessing Service annotated with service.spec.externalTrafficPolicy=Local. This feature works only with Weave IPAM (default).
IPTABLES_BACKEND - set to nft to use nftables backend for iptables (default is iptables)
```

#### Useful commands for test:
- find network plugin:
    - Run the command: `ps -aux | grep kubelet` and look at the configured `--network-plugin` flag.
- cni bin directory: `/opt/cni/bin`
- 

ipam-weave:



# Service Networking:
- service are accessible from all pods from all nodes
- service is only accessible within the cluster (type clusterIp)
- NodePort: it also exposes a port on all nodes in the cluster, so external users can access it
- service is just a virtual object, unlike pod, node etc
- kube proxy creates forwarding rule for each service so traffic is forwarded to pod:port
- kube proxy uses either `userspace`, `iptables` or `ipvs` for this. It's configured in kube-proxy param `--proxy-mode`. when not provided, its set to `iptables`.
- There's a range that services can use, its set by `kube-api-server` using argument `--service-cluster-ip-range`
- There's a pod range set as well. 
- pod and service range should not overlap. 

## Practice:
- see node ranges:
`ip addr`: look for cidr range of `eth0`
- see pod ranges:
    - look at cni options

# Cluster DNS:
- services name convention:
    - service.namespace.svc.cluster.local
    
- pod can have dns names if enabled, that case, pod dns will be ip with `-` instead of `.`. Everything else after first part follows same as svc.

# CoreDNS in kubernetes.
- study again if have time.
- coreDNS uses a config file called `/etc/coredns/CoreFile`
- its provided as configmap 
- There's a plugin called kubernetes in the conf file that handles kube dns stuff
- enabling pod dns is also in this file. 
- `/etc/resolve.conf` nameserver file is also defined here. 
- kubelet server configures dns server and ip. its provided as argument or in config file. 
- base domain is configured on this file as well. 

# Ingress:

