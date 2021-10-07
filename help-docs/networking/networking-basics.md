## Basics of networking:
* computers connect to network switch using network interface
- `ip link`: see interface (eth0 is normally the main interface, output is like this `<BROADCAST,MULTICAST,UP,LOWER_UP>`)
- Router connects many network together. 
    - it has two ips (internal and external)
- There can be many routers in a network. 
- Gateway (default gateway) is the main router that connects to internet
- `route`: see routing configs in computers. 
  - example output:
    ```
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
    default         _gateway        0.0.0.0         UG    100    0        0 eth0
    10.0.2.0        0.0.0.0         255.255.255.0   U     0      0        0 eth0
    _gateway        0.0.0.0         255.255.255.255 UH    100    0        0 eth0
    ```
    - when there's no route setup, the computer can't connect to any computer outside of the switch.

- `ip route add <destination-ip> via <router-ip>`: to add a route. 
    - Now we have to do that for all the computers in the network to all the networks on internet, which is not possible. 
    - Thats where the default gateway comes in. We can setup a default gateway which will be used for all connections
    - a default entry looks like this `default         _gateway        0.0.0.0         UG    100    0        0 eth0`
    - it can also look like this where "default" is changed to "0.0.0.0"
- `Gateway` field set to 0.0.0.0 means not gateway needed to connect to this ip.
- Another case can be that there are two routers in the network, one for internet and another for intranet.
    - in this case we will need two entries, one for internet, another for intranet
    
- when connecting local network via route command, make sure its connected both ways, from a>b and from b>a
- by default, packets are not forwarded from one interface to another.
    - it is defined in a file `/proc/sys/net/ipv4/ip_forward` which is set to 0
    - we can set it to one to enable packet forwarding
    - NOTE: setting it here won't persist it after reboot, we will need to change that to `/etc/sysctl.conf` file > `net.ipv4.ip_forward = 1`


## DNS:
- each computer have a dns resolution config file present at `/etc/resolv.conf` which has an entry `nameserver 127.0.0.53` for nameserver host.
- dns lookup order: `/etc/hosts` > local dns server> public dns server.
    - this order is defined in a file `/etc/nsswitch.conf`
    - `hosts:          files dns` this line shows the order. files is /etc/hosts and dns is dnsserver.
    - dnsserver itself points to public dns servers to lookup ips
#### important note on behavior of DNS:
- there's another entry in `/etc/resolv.conf` which is `search mycompany.com`
- This means that if i try a dns name with no domain, it will append `mycompany.com` to it. 
- the entry could have multiple domains like this. `search mycompany.com new.mycompany.com` and so on.
- `dig` is an alternate of `nslookup`

## Domain names:
- explanation: (www.google.com.)
    - . after com is there but not shown usually. It is the root
    - .com is the top level domain
    - google is the domain name for google
    - www is subdomain
    - same with maps.google.com, maps is subdomain.
    - we can create more subdomains even more levels.
- structure:
    - dns servers points to . (root)
    - root looks at .com domain
    - domain looks at google dns server. 
    - that provides ip for the website
    

CoreDNS: DNS server for kubernetes

