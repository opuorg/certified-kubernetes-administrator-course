# Practice Test - Multiple Schedulers
  - Take me to [Practice Test](https://kodekloud.com/courses/539883/lectures/9816597)

Notes:
- When creating custom scheduler, make sure to set --port argument to other then 0 or existing
- make sure to change leader-elect flag to false
- make sure to add --secure-port=0
- make to sure to change the healtchecks to new port
- change healthcheck scheam to http from https


- to custom schduler, add schedulerName: field under spec

Solutions to practice test - multiple schedulers
- Run the command 'kubectl get pods --namespace=kube-system'
  
  <details>

  ```
  $ kubectl get pods --namespace=kube-system
  ```
  </details>

- Run the command 'kubectl describe pod kube-scheduler-master --namespace=kube-system'

  <details>

  ```
  $ kubectl describe pod kube-scheduler-master --namespace=kube-system
  ```
  </details>

- Use the file at /etc/kubernetes/manifests/kube-scheduler.yaml to create your own scheduler. View answer file at /var/answers

  <details>

  ```
  $ kubectl create -f my-scheduler.yaml
  ```
  </details>

- Set schedulerName property on pod specification to the name of the new scheduler. File is located at /root/nginx-pod.yaml
  
  <details>

  ```
  master $ grep schedulerName /root/nginx-pod.yaml
  schedulerName: my-scheduler
  
  $ kubectl create -f /root/nginx-pod.yaml
  ```
  </details>








