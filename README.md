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
### NameSpace:
Create a NameSpace (ns):
```bash
kubectl create ns demo
#OR
kubectl create -f Namespace.yaml
kubectl get ns
```
### Pod:
Create a Pod (po):
```bash
kubectl run --generator=run-pod/v1 nginx --image=nginx:1.18 -l app.kubernetes.io/name=nginx --port=80 -n demo 
#OR
kubectl create -f Pod.yaml
kubectl get po -n demo
```

### initContainers:
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
Other sample (initContainers for deployment)
```bash
kubectl create -f initContainers-s2.yaml
kubectl get po -n demo
#Curl the pod's IP:
curl 10.32.0.5:80
#Output: <h1>Hello Minikube</h1>
```
TODO:I'll be back to it (After Deployment and volumes).


### Deployment:
Create a Deployment (deploy):
```bash
kubectl create deploy nginx-deployment --image nginx:1.18 --port=80 -n demo 
kubectl create -f Deployment.yaml
kubectl get deploy -n demo
```
Best Practices and Examples for Working with Kubernetes [Labels](https://www.replex.io/blog/9-best-practices-and-examples-for-working-with-kubernetes-labels)
,[Labels and Selectors](https://kubernetes-on-aws.readthedocs.io/en/latest/user-guide/labels.html)

New capabilities that we will have:
#### 1.Rollout:
```bash
kubectl rollout history deployment nginx-deployment
kubectl rollout history deployment nginx-deployment --revision=1
kubectl rollout history deployment nginx-deployment --revision=2
```
And then compare result.
Change the current state (revision):
To the previous version:
```bash
k rollout undo deployment nginx-deployment 
```
To the specific version:
```bash
k rollout undo deployment nginx-deployment --revision=2
```

You can add "revisionHistoryLimit" to your deployment:
```bash
.
.
replicas: 1
revisionHistoryLimit: 10 
.
.
```
It depends on number of comits per day.
As you saw "CHANGE-CAUSE" is empty; let's fill it via these ways:
```bash
kubectl create -f Deployment.yaml --record=true
kubectl rollout history deployment nginx-deployment
```
Your command has been saved as CHANGE-CAUSE.
Useful in Live objects:
```bash
kubectl set image deployment/nginx-deployment web=nginx:1.19 --record=true
kubectl rollout history deployment nginx-deployment
```

If you want to save changes from yaml files to CHANGE-CAUSE
You need to add this annotaion to yml file:
```bash
labels:
 app: nginx
annotations:
  kubernetes.io/change-cause: 'Changed nginx image to felan'
```
And then apply changes, we dont need to add --record=true

TODO:Test both(--record=true and annotation)

#### 2.Scaling:
Fix number of pods:
To scale up in Live object:
```bash
kubectl scale deployment/nginx-deployment --replicas=3
```
Autoscale (flexible):
Live Object:
```bash
kubectl autoscale deployment/nginx-deployment --min=1 --max=15  --cpu-percent=90
```
Output "hpa.autoscaling/nginx-deployment autoscaled."
Lets define HPA:
```bash
kubectl create -f HorizontalPodAutoscaler.yaml
```
take a look at out hpa yml file:
```bash
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-autoscalar
spec:
  maxReplicas: 15
  minReplicas: 1
  scaleTargetRef:
   apiVersion: apps/v1
   kind: Deployment
   name: nginx-deployment
  targetCPUUtilizationPercentage: 90
```
List of hpa:
```bash
kubectl get hpa
```


####  Role-Based Access Control -> RBAC
Objects:
1. ClusterRole (Group) - Cluster access  called => 2. Cluster RoleBinding
3. Role - NS - called => 4.RoleBinding

#### Create ns,sa,secret,role,rolebinding
```bash
cat ramin-sa.yaml
kubectl create -f ramin-sa.yaml
```
Now we need to create a KubeConfig for new user:
template of kubeConfig:
```bash
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data:  {cluster-ca}
    server:  {server-dns}
  name: {cluster-name}
contexts:
- context:
    cluster: {cluster-name}
    user: {user-name}
  name: {context-name}
current-context: {context-name}

kind: Config
users:
- name: {user-name}
  user:
    token: {secret-token}
```
Run this command to aim cluster-ca token: 
```bash
kubectl config view --flatten --minify 
```
cluster-ca,server-dns,cluster-name,context-name: same as root kubeconfig.
user-name: our new usename.
secret-token: to aim this token we have to run:
```bash 
kubectl get secrets -n qua
#Describe ramin-token-blah 
kubectl describe secret  ramin-token-fdcjx -n qua
```

Copy the user token and place in new kubeconfig secret-token.
Your new kubeconfig looks like:
```bash
cat config.yaml
```
Now you can test it before send it to new user:
```bash
kubectl --kubeconfig=config.yaml config use-context minikube
kubectl --kubeconfig=config.yaml get po -n qua
```

#