# k8s-workshop
Quick commands:
```bash
minikube start
```
Create NameSpace:
```bash
kubectl create ns demo
kubectl create -f 0.namepace-demo.yaml
```
Create Pod:
```bash
kubectl run nginx --image=nginx --restart=Never
kubectl create -f 1.pod-nginx.yaml
```

