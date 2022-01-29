# k8s-workshop
Quick commands:
```bash
minikube start
```
Create NameSpace:
```bash
kubectl create ns demo
#OR
kubectl create -f 0.namepace-demo.yaml
```
Create Pod:
```bash
kubectl run --generator=run-pod/v1 nginx --image=nginx:1.18 -l app.kubernetes.io/name=nginx --port=80 -n demo 
#OR
kubectl create -f 1.pod-nginx.yaml
```

