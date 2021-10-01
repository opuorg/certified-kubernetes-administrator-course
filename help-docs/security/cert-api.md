## cert api
- it is a CA service that can sign certs to add users to cluster
- normally it resides in master node
- master node has a url that can act as cert api who can sign certs
- process is like this:
1. user generate a key
   - use standard cert generation commands
    `openssl genrsa -out user.key 2048`
2. user then create s crs to sign 
   `openssl req -new-key user.key -subj '/CN=user' out user.csr`
   then email to admin
```
---
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: akshay
spec:
  groups:
  - system:authenticated
  usages:
  - digital signature
  - key encipherment
  - server auth
  signerName: kubernetes.io/kube-apiserver-client
  request: |
    base64-data
```
 
3. admin review it 
   `kubect get csr`
4. admin approve it, 
   `kubect certificate approve user`
5. Admin extract the cert and email to user 
    `kubect get csr user -o yaml`
   then decode and email to user
6. user can use it to auth

- For Bots, kubecroller manager do automated sign/approve etc
-  kubecroller manager has main cert passed as arguments

- delete csr:
  `kubectl delete csr agent-smith`