Docs: https://github.com/opuorg/kubernetes-the-hard-way

## Create/prep machines and create connectivity
1. Master nodes and worker nodes in same range. 
2. add public key of master nodes to other nodes to access connectivity between them.

## Generate Certificates
- Create a cert authority (master will act as a ca server as well)
- generate admin cert and sign with CA
- Generate the kube-controller-manager client certificates and sign with CA
- Generate the kube-proxy client certificate and sign with CA
- Generate the kube-scheduler client certificate and sign with CA
- Generate kube-apiserver certificate and sign with CA
  - This needs all the domains and ips that will be connecting to api server added as Alt Domains. Typically, they are
  ```
  DNS.1 = kubernetes
  DNS.2 = kubernetes.default
  DNS.3 = kubernetes.default.svc
  DNS.4 = kubernetes.default.svc.cluster.local
  IP.1 = 10.96.0.1
  IP.2 = 127.0.0.1
  IP.3 = <masternode>
  IP.4 = <wordernode>
  IP.5 = <wordernode>
  .....
  ```
- Generate ETCD server certificate and sign with CA
 - This needs all the domains and ips that will be connecting to etcd server added as Alt Domains. Typically, they are
   ```
   IP.2 = 127.0.0.1
   IP.3 = <masternode>
   IP.4 = <wordernode>
   IP.5 = <wordernode>
   .....
   ```
- Generate the service-account certificate and sign with CA 
- Copy the certs and keys to each master nodes.

## Generate kubeconfigs
- Generate kubeconfigs for `controller manager`, `kube-proxy`, `scheduler` and `admin` in master node. 
   - Find loadbalancer ip. Then export a var `export LOADBALANCER_ADDRESS=192.168.5.30`
   - Generate for `kube-proxy`
      ```
     {
      kubectl config set-cluster kubernetes-the-hard-way \
      --certificate-authority=ca.crt \
      --embed-certs=true \
      --server=https://${LOADBALANCER_ADDRESS}:6443 \
      --kubeconfig=kube-proxy.kubeconfig
      
      kubectl config set-credentials system:kube-proxy \
      --client-certificate=kube-proxy.crt \
      --client-key=kube-proxy.key \
      --embed-certs=true \
      --kubeconfig=kube-proxy.kubeconfig
      
      kubectl config set-context default \
      --cluster=kubernetes-the-hard-way \
      --user=system:kube-proxy \
      --kubeconfig=kube-proxy.kubeconfig
      
      kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
      }
     ```
   - generate for `kube-controller-manager`
   ```
   {
   kubectl config set-cluster kubernetes-the-hard-way \
   --certificate-authority=ca.crt \
   --embed-certs=true \
   --server=https://127.0.0.1:6443 \
   --kubeconfig=kube-controller-manager.kubeconfig
   
   kubectl config set-credentials system:kube-controller-manager \
   --client-certificate=kube-controller-manager.crt \
   --client-key=kube-controller-manager.key \
   --embed-certs=true \
   --kubeconfig=kube-controller-manager.kubeconfig
   
   kubectl config set-context default \
   --cluster=kubernetes-the-hard-way \
   --user=system:kube-controller-manager \
   --kubeconfig=kube-controller-manager.kubeconfig
   
   kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
   }
   ```
   - generate for `kube-scheduler`
   ```
   {
   kubectl config set-cluster kubernetes-the-hard-way \
   --certificate-authority=ca.crt \
   --embed-certs=true \
   --server=https://127.0.0.1:6443 \
   --kubeconfig=kube-scheduler.kubeconfig
   
   kubectl config set-credentials system:kube-scheduler \
   --client-certificate=kube-scheduler.crt \
   --client-key=kube-scheduler.key \
   --embed-certs=true \
   --kubeconfig=kube-scheduler.kubeconfig
   
   kubectl config set-context default \
   --cluster=kubernetes-the-hard-way \
   --user=system:kube-scheduler \
   --kubeconfig=kube-scheduler.kubeconfig
   
   kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
   }
   ```
   - Generate for `admin`
      - This could be in a different machine, in our case since we are using master as our admin, its in master.
   ```
   {
   kubectl config set-cluster kubernetes-the-hard-way \
   --certificate-authority=ca.crt \
   --embed-certs=true \
   --server=https://127.0.0.1:6443 \
   --kubeconfig=admin.kubeconfig
   
   kubectl config set-credentials admin \
   --client-certificate=admin.crt \
   --client-key=admin.key \
   --embed-certs=true \
   --kubeconfig=admin.kubeconfig
   
   kubectl config set-context default \
   --cluster=kubernetes-the-hard-way \
   --user=admin \
   --kubeconfig=admin.kubeconfig
   
   kubectl config use-context default --kubeconfig=admin.kubeconfig
   }
   ```
6. Copy the appropriate kube-proxy kubeconfig files to each worker instance:
```
for instance in worker-1 worker-2; do
  scp kube-proxy.kubeconfig ${instance}:~/
done
```
7. Copy the appropriate admin.kubeconfig, kube-controller-manager and kube-scheduler kubeconfig files to each controller instance:
```
for instance in master-1 master-2; do
  scp admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig ${instance}:~/
done
```

## Generate encryption keys
1. Generating the Data Encryption Config and Key
   - Generate an encryption key:
   ```
   ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
   ```
   - Create the encryption-config.yaml encryption config file:
   ```
   cat > encryption-config.yaml <<EOF
   kind: EncryptionConfig
   apiVersion: v1
   resources:
     - resources:
         - secrets
       providers:
         - aescbc:
             keys:
               - name: key1
                 secret: ${ENCRYPTION_KEY}
         - identity: {}
   EOF
   ```
   - Copy the encryption-config.yaml encryption config file to each controller instance:
   ```
   for instance in master-1 master-2; do
      scp encryption-config.yaml ${instance}:~/
   done
   ```
   - Move encryption-config.yaml encryption config file to appropriate directory in each controller
   `sudo mv encryption-config.yaml /var/lib/kubernetes/`
      - if the directory not exist, create `sudo mkdir /var/lib/kubernetes`
   
## Bootstrapping the etcd Cluster

Note: The commands in this lab must be run on each controller instance: master-1, and master-2. Login to each of these using an SSH terminal.
1. Download the official etcd release binaries from the coreos/etcd GitHub project:
```
wget -q --show-progress --https-only --timestamping \
  "https://github.com/coreos/etcd/releases/download/v3.3.9/etcd-v3.3.9-linux-amd64.tar.gz"
```
2. Extract and install the etcd server and the etcdctl command line utility:
```
{
  tar -xvf etcd-v3.3.9-linux-amd64.tar.gz
  sudo mv etcd-v3.3.9-linux-amd64/etcd* /usr/local/bin/
}
```
3. Configure the etcd Server
```
{
  sudo mkdir -p /etc/etcd /var/lib/etcd
  sudo cp ca.crt etcd-server.key etcd-server.crt /etc/etcd/
}
```
   - The instance internal IP address will be used to serve client requests and communicate with etcd cluster peers. Retrieve the internal IP address of the master(etcd) nodes:
```
INTERNAL_IP=$(ip addr show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f 1)
```
   - Each etcd member must have a unique name within an etcd cluster. Set the etcd name to match the hostname of the current compute instance:
`ETCD_NAME=$(hostname -s)`

4. Create the etcd.service systemd unit file:
```
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/etcd-server.crt \\
  --key-file=/etc/etcd/etcd-server.key \\
  --peer-cert-file=/etc/etcd/etcd-server.crt \\
  --peer-key-file=/etc/etcd/etcd-server.key \\
  --trusted-ca-file=/etc/etcd/ca.crt \\
  --peer-trusted-ca-file=/etc/etcd/ca.crt \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster master-1=https://192.168.5.11:2380,master-2=https://192.168.5.12:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
- Start the etcd Server
```
{
  sudo systemctl daemon-reload
  sudo systemctl enable etcd
  sudo systemctl start etcd
}
```
- Verification: List the etcd cluster members:
```
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.crt \
  --cert=/etc/etcd/etcd-server.crt \
  --key=/etc/etcd/etcd-server.key
```
   - ouput should be like this
```
45bf9ccad8d8900a, started, master-2, https://192.168.5.12:2380, https://192.168.5.12:2379
54a5796a6803f252, started, master-1, https://192.168.5.11:2380, https://192.168.5.11:2379
```
## Bootstrapping the Kubernetes Control Plane
