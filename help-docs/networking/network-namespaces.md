## Network namespaces:
- namespaces are different for each container
- but host has its own namespace which has visibility to all container ns
- a process inside container is same inside the host but with different id


- containers have their own virtual interface, routing table, arp table etc

- `ip netns add <name>`: add a network namespace in linux
- `ip netns`: list ns
- `ip link`: list interfaces in hosts
- `ip netns exec <name> ip link`: see list of interfaces inside a namespace
- `ip -n <name> link`: same as above command. [Still not clear]
- same can be done for `arp`, `route` commands
  `ip netns exec <name> route`
- Just like we can connect machines via commands, we can connect namespaces the same way via a virtual cable (pipe)
    - in order to do that, we will have to create a veth first. 
        - `ip link add <veth-red1> type veth peer name <veth-blue1>`
    - now attach each side to containers
        -  `ip link set veth-red1 netns red` and `ip link set veth-blue1 netns blue`
    - now assign ip address to each veth.
        - `ip netns exec red ip addr add 192.168.15.1 dev veth-red1` and `ip netns exec blue ip addr add 192.168.15.2 dev veth-blue1`
        - now bring up each eth using commands `ip netns exec red ip link set veth-red1 up` and `ip netns exec blue ip link set veth-blue1 up`
    
- Now connecting each via interfaces is not possible for lot of machines. For that we need to use a software like `linux bridge`
    - `ip link add <v-net-0> type bridge` Create a bridge int
    - by default its down. 
    - `ip link set dev v-net-0 up`
    
- Now we need to delete the veth link we created before. 
    - `ip netns exec red ip link del veth-red1` to delete link. 
    - Deleting from one side also deletes from other side as well.
    
- Now create another veth (virtual cable) 
    - `ip link add <veth-red> type veth peer name <veth-red-br>`
        - note that the other side is not named to connect to other ns, but the bridge. (just the naming convention here, we will attach later)
    - repeat the same for blue network
- Now connect each side with netns and bridge
    - `ip link set veth-red netns red`: connect one side to red. 
    - `ip link set veth-red-br master v-net-0`: connect other side to bridge (master)
    - repeat same for other ns
    
- Now set ip address for each eth
    - `ip netns exec red ip addr add 192.168.15.1 dev veth-red`
    - `ip netns exec blue ip addr add 192.168.15.2 dev veth-blue`
    
- Bring them up. 
    - `ip netns exec red ip link set dev veth-red up`
    - `ip netns exec blue ip link set dev veth-blue up`
    
- Add an ip to bridge interface:
    - `ip addr add 192.168.15.5/24 dev v-net-0`
    
- Now we need to add bridge as gateway for the namespaces to connect to the localhost (since that is connected to both ns and localhost)
    - `ip netns exec blue ip route add 192.168.1.0/24 via 192.168.15.5`
- now by default, packet is not fowarded 
    - `iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -j MASQUERADE`
    
- now we add a default gateway
    - `ip netns exec blue ip route add default via 192.168.15.5`
    
- But any traffic from ouside internet can't reach our ns. for that, we need to setup port forwarding.
    - `$ iptables -t nat -A PREROUTING --dport 80 --to-destination 192.168.15.2:80 -j DNAT`
    
Notes: 
```
While testing the Network Namespaces, if you come across issues where you can’t ping one namespace from the other, make sure you set the NETMASK while setting IP Address. ie: 192.168.1.10/24
ip -n red addr add 192.168.1.10/24 dev veth-red

Another thing to check is FirewallD/IP Table rules. Either add rules to IP Tables to allow traffic from one namespace to another. Or disable IP Tables all together (Only in a learning environment).
```