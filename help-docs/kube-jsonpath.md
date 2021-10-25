Jsonpath:
```
kubectl get no -o=jsonpath='{.apiVersion}'
kubectl get no -o=jsonpath='{.items[*].apiVersion}-{.apiVersion}'
kubectl get no -o=jsonpath='{.items[*].apiVersion}{'\n}{.apiVersion}'
kubectl get no -o=jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.capacity.cpu}{"\n"}{end}'
```

Custom Column:
```
kubectl get no -o=custom-columns=NODE:.metadata.name,CPU:.status.capacity.cpu
```

Sort:
```
kubectl get no --sort-by=.status.capacity.cpu
```