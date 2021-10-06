## ingresses:
- GCE
- Nginx
- Coutour
- HAProxy
- Traefik
- Istio

- Only GCE and NGINX are maintained and supported by Kubernetes

#### Nginx:
- Very basic things we need for a ingress to work:
    - nginx deployment.
    - nginx configmap: to hold the configs
    - serviceaccount: for permissions
    - service: for exposing the nodeport
    - a `Default backend` need to be configured as well when there's no rule match. 
    
- Next we need to create rules to forward traffic. That is called ingress service which governs which path to be directed to where. 
    - This is deployed for each service/rules/urls
    
1. deployment yaml:
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ingress-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      name: nginx-ingress
  template:
    metadata:
      labels:
        name: nginx-ingress
    spec:
      serviceAccountName: ingress-serviceaccount
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0
          args:
            - /nginx-ingress-controller
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - name: http
              containerPort: 80
            - name: https
              containerPort: 443
```

2. Create configmap
```
kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-configuration
```

3. Create seviceaccount
```
kubectl create -f ingress-sa.yaml
```

4. Create Service:
```
apiVersion: v1
kind: Service
metadata:
  name: ingress
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  - port: 443
    targetPort: 443
    protocol: TCP
    name: https
  selector:
    name: nginx-ingress
```

- Now we create rules for ingress. Few examples are below.

- 1 Rule and 2 Paths.

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  rules:
  - http:
      paths:
      - path: /wear
        pathType: Prefix
        backend:
          service:
            name: wear-service
            port: 
              number: 80
      - path: /watch
        pathType: Prefix
        backend:
          service:
            name: watch-service
            port: 
              number: 80
```
- Describe the earlier created ingress resource

```
$ kubectl describe ingress ingress-wear-watch
Name:             ingress-wear-watch
Namespace:        default
Address:
Default backend:  default-http-backend:80 (<none>)
Rules:
  Host        Path  Backends
  ----        ----  --------
  *
              /wear    wear-service:80 (<none>)
              /watch   watch-service:80 (<none>)
Annotations:  <none>
Events:
  Type    Reason  Age   From                      Message
  ----    ------  ----  ----                      -------
  Normal  CREATE  23s   nginx-ingress-controller  Ingress default/ingress-wear-watch

```

- 2 Rules and 1 Path each.
```
# Ingress-wear-watch.yaml

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  rules:
  - host: wear.my-online-store.com
    http:
      paths:
      - backend:
          serviceName: wear-service
          servicePort: 80
  - host: watch.my-online-store.com
    http:
      paths:
      - backend:
          serviceName: watch-service
          servicePort: 80
```
- It can also be done by declaritve commands
```
Format - kubectl create ingress <ingress-name> --rule="host/path=service:port"

Example - kubectl create ingress ingress-test --rule="wear.my-online-store.com/wear*=wear-service:80"
```

#### References Docs
- https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-ingress-em-

- https://kubernetes.io/docs/concepts/services-networking/ingress/
- https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/
- https://thenewstack.io/kubernetes-ingress-for-beginners/

## Annotations and rewrite-target
- Nginx rules/options reference: https://kubernetes.github.io/ingress-nginx/examples/

- Reason:
  - We have watch and wear service deploy at these urls: `http://<watch-service>:<port>/` and `http://<wear-service>:<port>/`
  - With ingress we are trying to achieve is this. 
    `http://<ingress-service>:<ingress-port>/watch –> http://<watch-service>:<port>/` and  `http://<ingress-service>:<ingress-port>/wear –> http://<wear-service>:<port>/`
  - when we don't have rewrite rule, the urls will be like this.
  `http://<ingress-service>:<ingress-port>/watch –> http://<watch-service>:<port>/watch` and `http://<ingress-service>:<ingress-port>/wear –> http://<wear-service>:<port>/wear`
    Notice the /watch and /wear at the url.
  - Now this extra url is not configured in our service apps. This is why we need rewrite. 
  - syntax is `replace(path, rewrite-target)`. This rewrites the URL by replacing whatever is under `rules->http->paths->path`
  - example below:
  
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /pay
        backend:
          serviceName: pay-service
          servicePort: 8282
```
  
- Another example below.
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  name: rewrite
  namespace: default
spec:
  rules:
  - host: rewrite.bar.com
    http:
      paths:
      - backend:
          serviceName: http-svc
          servicePort: 80
        path: /something(/|$)(.*)
```
In this ingress definition, any characters captured by (.*) will be assigned to the placeholder $2, which is then used as a parameter in the rewrite-target annotation.

For example, the ingress definition above will result in the following rewrites:
```
rewrite.bar.com/something rewrites to rewrite.bar.com/
rewrite.bar.com/something/ rewrites to rewrite.bar.com/
rewrite.bar.com/something/new rewrites to rewrite.bar.com/new
```

# notes:
- careful about the ports, it should point to the service `TargetPort:` not `Endpoints:` port
- `nodePort`: need to be setup in yaml if mentioned
