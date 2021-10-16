
defininition-file basics:

```
apiVersion: 
kind: 
metadata:
    name:
    labels:
        key: value
        key:value
spec:
```

- replicaset (replication controller before):
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
```
- template section defines when creating new pod
- selector adds existing pods by label.
