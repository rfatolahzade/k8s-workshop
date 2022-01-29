![Adventures in Kubernetes](https://miro.medium.com/max/1100/1*9B8HEd8Ap3vVL126UaFldQ.png)
# k8s-workshop
Quick commands:
```bash
minikube start
```
Four required fields:
```bash
apiVersion    # kubectl api-resources
Kind
Metadata
spec
```
Aliases:
```bash
cat <<EOF >> ~/.bashrc
alias k='kubectl'
alias kga='watch -x kubectl get all -o wide'
alias kgad='watch -dx kubectl get all -o wide'
alias kcf='kubectl create -f'
alias wk='watch -x kubectl'
alias wkd='watch -dx kubectl'
alias kd='kubectl delete'
alias kcc='k config current-context'
alias kcu='k config use-context'
alias kg='k get'
alias kdp='k describe pod' 
alias kdes='k describe'
alias kdd='k describe deployment'
alias kds='k describe svc'
alias kdr='k describe replicaset'
#alias kk='k3s kubectl'
alias vk='k --kubeconfig'
alias kcg='k config get-contexts'
alias kgaks='watch -x kubectl get all -o wide -n kube-system'
alias kapi='kubectl api-resources'

EOF

 ``` 
 
# Adventure time

Create a NameSpace (ns):
```bash
kubectl create ns demo
#OR
kubectl create -f 0.namepace-demo.yaml
kubectl get ns
```
Create a Pod (po):
```bash
kubectl run --generator=run-pod/v1 nginx --image=nginx:1.18 -l app.kubernetes.io/name=nginx --port=80 -n demo 
#OR
kubectl create -f 1.pod-nginx.yaml
kubectl get po -n demo
```

