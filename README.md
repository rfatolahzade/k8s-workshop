![Adventures in Kubernetes](https://miro.medium.com/max/1100/1*9B8HEd8Ap3vVL126UaFldQ.png)
# k8s-workshop
Bash completion:
```bash
apt-get install bash-completion
source /usr/share/bash-completion/bash_completion
echo 'source <(kubectl completion bash)' >>~/.bashrc
#echo 'alias k=kubectl' >>~/.bashrc
echo 'complete -F __start_kubectl k' >>~/.bashrc
```
You os Distributor ID and Release:
```bash
lsb_release -a
```
In Docker Client and daemon communicate using a REST API, over UNIX sockets or a network interface.
In Kubernetes :
Master node contains: API SERVER- KUBE CONTROL MANAGER - ETCD - KUBELET- SCHEDULER (CLOUD CONTROLL MANAGER)
API SERVER (Kube API): similar to REST API in Docker -Get method. 
- KUBE CONTROL MANAGER :  Brain(has agents) - algorithm,self-healing-scale up ..check env  managed of all components
- ETCD: No-SQL database(Key-Value) (details of all nodes,pods options,names,volume,ports,configuration,additional things -HA ETCD  --- Backup able (Velero) ~ETCD can outter component
- KUBELET: image pull, volume mount , config ,expose port call network, self-healing , controller , up the pod, launch components of kube.
- SCHEDULER: Only SCHEDULING, where we can able deploy pod on which node, usage CPU/Memory of nodes needs,decider to deploy where to deploy. 
- CLOUD CONTROLL MANAGER: AWS,GCP,OpenStack optional,extra feature,connect to Cloud API Provider.connect to our cloud LoadBlancer.
Worker node contains:: KUBELET,KubeProxy
KubeProxy: network layer,Connection between containers,assigning IP,Connection between components.


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
NAMESPACE: Network Scope (Except Node,PV - except these components to call all other components we have to set -n )
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
kubectl run --generator=run-pod/v1 nginx --image=nginx:1.18 -l app.kubernetes.io/name=nginx --port=80  
#OR
kubectl create -f Pod.yaml
kubectl get po 
```
Pod to Pod needs CNI.
### initContainers:
Create a Pod with an initContainers,You can specify init containers in the Pod specification alongside the "containers" array (which describes app containers)
The init containers: specialized containers that run before app containers in a Pod. Init containers can contain utilities or setup scripts not present in an app image.
InitContainers (Inner a pod) can see each other.
InitContainers start firt (create a file, echo some settings,check something... and then delete)
InitContainers Its better to have seperated initcontainers for each job, if one task down we can find out.
First Debugging step: Emphereral Container | SideCar : tools and then you have to debug the LO or Svc or ..
After apply a pod, we can just change these: spec.containers[*].image (version),spec.initContainers[*].image (version),spec.activeDeadlineSeconds or spec.tolerations.
activeDeadlineSeconds : kill the container after this time .
Restart Policy for pods(When Error Happened): Always (Exit code 0 or 1 restart the pod), OnFailure (try to catch exit code 0), Never (at all)


```bash
kubectl create -f initContainers.yaml
kubectl get po 
```
Let's take a look at the description of our pod:

```bash
kubectl describe pod/nginx 
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
kubectl logs nginx  -c my-init-containers
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
kubectl get po 
#Curl the pod's IP:
curl 10.32.0.5:80
#Output: <h1>Hello Minikube</h1>
```
TODO:I'll be back to it (After Deployment and volumes).

```bash
LOGS: -f (follow)
#DEFAULT
k logs -f nginx-adv     
k logs -f nginx-adv -c web
#INIT
k logs -f nginx-adv -c my-init-containers
#DB COnatainer
k logs -f nginx-adv -c db
k exec -it nginx-adv -c db -- sh
mysql -u root -p
show databases;

EXEC:
k  exec -it nginx-adv -- sh
curl localhost

OUTPUT:
k  get po nginx-adv -o yaml > nginx-temp.yaml
```

#WORKLOADS (Pods controllers-managers):
Containers Deployments (Workloads):
1. Replicasets (Scale up-down)--- Replication Controller (limits label-rollbacking-probes)
2. Deployments (NO limits for label-rollbacking-probes-Replication Controller) - History (rollback)
3. StatefulSets
4. DaemonSet 
5. Jobs
6. CronJob

### Deployment:
Useful for stateless applicaions.
Create a Deployment (deploy):
```bash
kubectl create deploy nginx-deployment --image nginx:1.18 --port=80  
kubectl create -f Deployment.yaml
kubectl get deploy 
```
Manage who has this label (app: nginx) Also labels of deployments useful for Services.

Best Practices and Examples for Working with Kubernetes [Labels](https://www.replex.io/blog/9-best-practices-and-examples-for-working-with-kubernetes-labels)
,[Labels and Selectors](https://kubernetes-on-aws.readthedocs.io/en/latest/user-guide/labels.html)
```bash
SAMPLE LABELS:
key-value for user
app.kubernetes.io/name         mysql
app.kubernetes.io/instance     mysql-abcxzy
app.kubernetes.io/verion       1.0.1
app.kubernetes.io/component    database 
app.kubernetes.io/part-of      wordpress
app.kubernetes.io/managed-by   helm
app.kubernetes.io/created-by   controller-manager
```
Sample pod manifest that used a label:
```bash
kind: pod
.
.
.
spec:
  nodeSelector: 
  accelerator: nvidia-p100
```
Force to deploy on this worker that had this label. If doesnt exist,no deploy
```bash
k get pods -l x=y,z=1
```
some sample:
```bash
kg deploy -l app=nginx
kg po -l app=nginx
#Version (rollback-history - option of wrokloads)
k rollout history deploy nginx-deployment 
k rollout history deploy nginx-deployment  --revision=1 #nginx:1.18
k rollout history deploy nginx-deployment  --revision=2  #nginx:1.19
Current:  19

k rollout undo deploy nginx-deployment 
k rollout history deploy nginx-deployment 
k rollout history deploy nginx-deployment  --revision=2  #nginx:1.19
k rollout history deploy nginx-deployment  --revision=3  #nginx:1.18

k rollout undo deploy nginx-deployment

k set image deployment/nginx-deployment web=nginx:1.19 --record
Only use on liveObject
these rev saved on ETCD (Rollback in app not db)
#Scale HARD Commandly:
k scale deployment/nginx-deployment --replicas=4

#AutoSCALE Commandly:
minikube addons enable metrics-server
#Sum of All deployment workload CPU usage
k autoscale deployment/nginx-deployment --min=2 --max=10 --cpu-percent=80
kg hpa


#Pause The APP(pod- workloads):
k rollout pause deploy nginx-deployment 
k rollout resume deploy nginx-deployment 
kdes po nginx
#Canary Deployment:
stable --> other instance  new version.
paused: true
Canary + ZERO DOWN TIME
```

#### Container Probe:
Handlers:
- ExecAction
- TCPSocketAction
- HTTPGetAction

OUTPUTS or Probes: 
Success : Such as  TCPSocketAction port 3306 up
Failure : Such as called "/" and 404 not found ( app is up but / emppty) all numbers except 2XX 3XX
unknown : Such as Probes didnt run

Liveness probes: it has to show container is up pod state is running ( ram cpu to up the pod, pv, pvc ...)
Readiness probes: ( After container created and started then --> handle response to requests) service inner container. check just "/" without example.com just localhost , HTTP get Proxy 
Startup probes : time to run app (jar file, binary file ...) - after this  active Readiness probe -is Self-Healing , periodSeconds: 5  open the 80 port( tcpSocket.port: 80) for five times, failureThreshold: 2 if telent failed its ok.  successThreshold: 2 our app has to sucessed 2 times. and then other probes

If failure happend Container restart Policy check it out.

STRATEGY: Delete old up the new :RollingUpdate (default) 
Other type Recreate:it has a down-time useful for database  single .read in the directory , new and old couldnt write in same dir 


maxSurge : after delete a pod how many instance should be up.
maxUnavailable : max of containers can be Unavailable such as set 3 --> step1: kill 3 old create 3 new do it 3 times.
 25% or count of pod
 
 ```bash
maxUnavailable: 0
# IF you have pods for update: new users go to new version - count of pods will be same as past when you want to update
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
```
ZERO down-time Deployment: Pause + Define Probes + Strategy maxUnavailable: 0
#### Manage resources:
 ```bash
resources:
  requests:          #Startup Reserve 
     memory: "64Mi"
	 cpu: "250m"     # 1000m = 1 CPU
  limits:
     memory: "128Mi"  #using limits ,max of req, kube control manager manage it, limit for container.
	 cpu: "500m"
 ```
 #Best --> Monitoring tools for set these limits

#### ANNOTATIONS
Similar to labels but belong to your app. key-value for your app action will be happen based on these ANNOTATIONS.
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


#### StatefulSets
Pre state is important,connection between pods,where was up,
feature: ordered by , volumes remains,Service Headless service(CLUSTER-IP is None, no network layer, point-to-point connection between containers, without lataency).via this service sts findout other pods.
podManagementPolicy: OrderedReady, Prallel->(looks like deployment)

StatefulSets: such as Master-ReadReplicas app ( new nodes have to carry on previous state )
#Strategy:
On Delete
Rolling Update
- Partitions

If you set 
  updateStrategy:
    type: OnDelete
Your changes doesnt apply till you delete previuos pods:
Delete a pod (any pod):
kd pod/nginx-6
now your pod is updated to new version.
Both version is up.


If you set 
  updateStrategy:
    type: RollingUpdate
 Now self delete on update happen.
 
Set Partition:
  updateStrategy:
    rollingUpdate:
       partition: 2  #default 0
two pod will not update


Canary: RollingUpdate + Probe
BlueGreen: Service endpoint change (Live to Stage) -Same NameSpace
Self-Healing  (Canary-BlueGreen)

#### DaemonSet:
any node that we have or new added node have to have our pod. (usefull for log management app, Exporters,)
DaemonSet has two update strategy types:
OnDelete same as sts  ( you have to delete pod to update)
RollingUpdate same as deployment (maxSurage-maxUnavailable)
    OnDelete: With OnDelete update strategy, after you update a DaemonSet template, new DaemonSet pods will only be created when you manually delete old DaemonSet pods. This is the same behavior of DaemonSet in Kubernetes version 1.5 or before.
    RollingUpdate: This is the default update strategy.
    With RollingUpdate update strategy, after you update a DaemonSet template, old DaemonSet pods will be killed, and new DaemonSet pods will be created automatically, in a controlled fashion. At most one pod of the DaemonSet will be running on each node during the whole update process.
	
	#### Job
Time or task is important. how many task run. no probe no update stratgy
Default non-prallel 
 k logs pod shows our resultsets of command
There is no interval time/schedule  = > CronJob: same as job + schedule
Pay attention to kube-control-manager time

concurrencyPolicy default is Allow (define what will happen after job created is specific time) - Other value:Forbid,Replace
Allow: interval 0  is running and after time up interval 1 start.
Forbid: if interval 0 didnt finish scape interval time 2 , conflict between jobs solve.
Replace: if interval 0 didnt  done , interval 1 replace on 0

History in CronJobs:
successfulJobsHistoryLimit -default 3 and failedJobsHistoryLimit - default 1

All jobs can be suspend (true,false)


#### SERVICE 
(app layer -- types:), LoadBlancer (MetalLB, udf,http,https,..)Outter cluster,INGRESS Controller(just http,https handler, multi directive)Outter cluster, Networking (Management k8s components - calico,flannel ,..)
svc:Connection between pods and contaners our apps inside cluster layer( same ns or diff)

kind : Service ( connetion inside cluster layer: between contaiers/pods,between pod cross over ns,pod to out cluster,service to internet)
S -> Pod (to call the pod we use service) : selector.app: myapp (label)
type empty = clusetrIP
port: 80  
targetPort: 9376   port pod -container (myapp) natting (9376 -> 80)  -- config of app you can not change 80 for nginx  , to change you have to change nginx config.
Features : natting in svc.

If you want to use Multi-Port Service you have to set name for them in your manifest.
- Headles Service : ClusterIP : "None"
Useful for Primary-Sec in deploy DB. Between Pods.
Useful for make connection between outter apps to  cluster. such as sms gateway.External Services , External database(Other cluster) 

- ClusterIP: default type. There is no clusterIP. if ClusterIP:"None" then headless service.
Type ClusterIP and Headles access able in cluster.

- NodePort: 
Useful for connection between LB (cloud LB)
Useful for test app directly.
-port: 80
 targetPort: 80
 nodePort: 30004

- LoadBlancer: 
It works when you have a LoadBlancer.
If you have had multi LoadBlancer you have to set IP to manifest:
status:
  loadBlancer:
    ingress:
	- ip : 192.0.2.130
	
- ExternalName:
Outter cluster : You have to set externalName: my.databas.example.com
Outter NS in this case (prod): my-service1.prod.svs.cluster.local
- External IPs: same as ExternalName but set IP, targetPort (ther port of app)


-Ingress:
Needs ingress controller.
```bash
kg netpol

k get ep -o wide
minikube tunnel
minikube service nginx --url
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
#### NetworkPolicy
Sample NetworkPolicy to limit access to app:
We defined policyTypes,ipBlock,namespaceSelector,podselector,limit access from range of ports
```bash
cat networkpolicy.yaml
```

#### pv-pvc hostpath
```bash
cat pv-pvc.yaml
cat deployment.yaml
```

