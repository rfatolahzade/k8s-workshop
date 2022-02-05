![Adventures in Kubernetes](https://miro.medium.com/max/1100/1*9B8HEd8Ap3vVL126UaFldQ.png)
# k8s-workshop
Quick commands:
```bash
minikube start
```
Four required fields:
```bash
apiVersion    # kubectl api-resources
kind          # Namespace,pod,deployment,StatefulSets,Service,PersistentVolume,PersistentVolumeClaim,...
metadata      # name,label,namespace,annotations,...
spec          # replicas,containers,selector,...
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
#### NameSpace
Create a NameSpace (ns):
```bash
kubectl create ns demo
#OR
kubectl create -f Namespace.yaml
kubectl get ns
```
#### Pod
Create a Pod (po):
```bash
kubectl run --generator=run-pod/v1 nginx --image=nginx:1.18 -l app.kubernetes.io/name=nginx --port=80 -n demo 
#OR
kubectl create -f Pod.yaml
kubectl get po -n demo
```

#### initContainers
Create a Pod with an initContainers,You can specify init containers in the Pod specification alongside the "containers" array (which describes app containers)
The init containers: specialized containers that run before app containers in a Pod. Init containers can contain utilities or setup scripts not present in an app image.

```bash
kubectl create -f initContainers.yaml
kubectl get po -n demo
```
Let's take a look at the description of our pod:

```bash
kubectl describe pod/nginx -n demo
```
As you can see our lightweight Init Containers that are named my-init-containers ran first:
```bash
Init Containers:
  my-init-containers:
    Container ID:  docker://c46bd4ac5b6f2a4bb19cd5c68b238321a576ab9a50004d54c089d5adc907e582
    Image:         busybox
```
And the state of the command is Completed:
```bash
    Command:
      /bin/sh
    Args:
      -c
      echo 'Hello From initContainers' 
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
```
In the Events took steps was:
```bash
Events:
  Type    Reason     Age   From                   Message
  ----    ------     ----  ----                   -------
  Normal  Scheduled  73s   default-scheduler      Successfully assigned demo/nginx to controlplane
  Normal  Pulling    70s   kubelet, controlplane  Pulling image "busybox"
  Normal  Pulled     67s   kubelet, controlplane  Successfully pulled image "busybox"
  Normal  Created    67s   kubelet, controlplane  Created container my-init-containers
  Normal  Started    67s   kubelet, controlplane  Started container my-init-containers
  Normal  Pulling    66s   kubelet, controlplane  Pulling image "nginx:1.18"
  Normal  Pulled     55s   kubelet, controlplane  Successfully pulled image "nginx:1.18"
  Normal  Created    55s   kubelet, controlplane  Created container web
  Normal  Started    54s   kubelet, controlplane  Started container web
```
Accessing logs from Init Containers,Pass the Init Container name along with the Pod name to access its logs.
To Inspect the first(only) init container:
```bash
#kubectl logs <pod-name> -c <init-container>
kubectl logs nginx -n demo -c my-init-containers
```
Output:
```bash
Hello From initContainers
```

```diff
- NOTICE 1: Init containers always run to completion.
- NOTICE 2: Each init container must complete successfully before the next one starts.
- NOTICE 3: init containers do not support lifecycle, livenessProbe, readinessProbe, or startupProbe because they must run to completion before the Pod can be ready.
```