## Manipulate file:
cat /etc/passwd | cut -f 1 -d ':' > /etc/foo/passwd

	1. Cat /etc/passwd
	2. Remove first section of each line having a delimiter : 
	3. Send the output to /etc/foo/passwd
 
## Copy files from and to pod
kubectl cp test/busy:/etc/foo/passwd ./passwd
kubectl cp <some-namespace>/<some-pod>:/tmp/foo /tmp/bar


## Jsonpath
```
kubectl get nodes -o=jsonpath='{.items[*].metadata.name}' > /opt/outputs/node_names.txt
kubectl get nodes -o=jsonpath='{range .items[*]} {.metadata.name} {"\t"} {.status.capacity.cpu} {"\n"} {end}'
kubectl get nodes -o jsonpath='{range .items[*]} {.metadata.name} {"\t"} {.status.nodeInfo.osImage} {"\n"} {end} '
kubectl get node node01 -o json > /opt/outputs/node01.json
kubectl get nodes -o jsonpath='{.items[*].status.nodeInfo.osImage}' > /opt/outputs/nodes_os.txt
kubectl config view --kubeconfig=my-kube-config -o jsonpath="{.users[*].name}" > /opt/outputs/users.txt
kubectl get pv --sort-by=.spec.capacity.storage > /opt/outputs/storage-capacity-sorted.txt
kubectl get pv --sort-by=.spec.capacity.storage -o=custom-columns=NAME:.metadata.name,CAPACITY:.spec.capacity.storage > /opt/outputs/pv-and-capacity-sorted.txt
kubectl get deployment -n admin2406 -o=custom-columns="DEPLOYMENT:.metadata.name,CONTAINER_IMAGE:.spec.template.spec.containers[*].image,READY_REPLICAS:.spec.replicas,NAMESPACE:.metadata.namespace"
```

## Manipulate Pod
Annotate based on labels:
```
k -n sun annotate pod -l protected=true protected="do not delete this pod"
```

Label based on label (every pod with label type worker will receive the following label protect true)
```
k -n sun label pod -l type=worker protected=true
```

How to Label a node
```
kubectl label nodes node01 color=blue
```

Read logs from an internal drive instead of stdout
```
kubectl exec webapp -- cat /log/app.log
```

## Check API values
```
kubectl explain pods --recursive | less
```

## TMP Test Pod or Service
```
k run tmp --restart=Never --rm -i --image=nginx:alpine -- curl -m 5 10.12.2.15
k run tmp --restart=Never --rm -i --image=nginx:alpine -- curl -m 5 sun-srv.sun:9999
k -n venus run tmp --restart=Never --rm -i --image=busybox -i -- wget -O- frontend:80 
```
```
kubectl run -it --rm test --image=busybox --restart=Never -- sh
nc -zv serviceName servicePort
nc -zv secure-service 80
```

## POD / Service DNS
POD
Without service: 
172-17-0-3.default.pod.cluster.local

With Service EP:
pod-ip-address.service-name.my-namespace.svc.cluster-domain.example

SVC
Short: manager-api-svc.mars
Long : manager-api-svc.mars.svc.cluster.local 


## Docker and Podman
```
sudo podman build -t registry.killer.sh:5000/sun-cipher:latest -t registry.killer.sh:5000/sun-cipher:v1-docker .
sudo docker push registry.killer.sh:5000/sun-cipher:v1-docker
sudo docker run -d --name=test registry.killer.sh:5000/sun-cipher:v1-docker
sudo docker ps
sudo docker logs test
sudo docker save nginx:latest > nginx.tar

docker history image
docker run python:3.6 cat /etc/*release*
docker run -p 8383:8080 webapp-color:lite
```

## Helm 
```
helm search repo bitnami 
helm search repo bitnami --versions
helm upgrade internal-issue-report-apiv2 bitnami/nginx -n mercury
helm show chart ingress-nginx/ingress-nginx
helm pull --untar ingress-nginx/ingress-nginx
helm create mychart
helm install mychart1 ./mychart
```

## Liveness, readiness and startup probe
### Liveness
The kubelet uses liveness probes to know when to restart a container. For example, liveness probes could catch a deadlock, where an application is running, but unable to make progress. Restarting a container in such a state can help to make the application more available despite bugs.
From <https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/> 


### Readiness
The kubelet uses readiness probes to know when a container is ready to start accepting traffic. A Pod is considered ready when all of its containers are ready. One use of this signal is to control which Pods are used as backends for Services. When a Pod is not ready, it is removed from Service load balancers.
From <https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/> 


### Startup 
The kubelet uses startup probes to know when a container application has started. If such a probe is configured, it disables liveness and readiness checks until it succeeds, making sure those probes don't interfere with the application startup. This can be used to adopt liveness checks on slow starting containers, avoiding them getting killed by the kubelet before they are up and running.

From <https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/> 


## Volumes
 ### hostPath  	
 ```
  volumes:
  - name: test-volume
    hostPath:
      path: /data
      type: Directory	  
```

### PVC	
```
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: task-pv-claim	  
```

### emptyDir
```
volumes:
  - name: cache-volume
    emptyDir: {}
```

## Admission controller
Check enable-admission-plugins

	• ps -ef | grepkube-apiserver | grep admission-plugins
	• kubectl describe kube-apiserver-controlplane -n kube-system 
	• grep enable-admission-plugins /etc/kubernetes/manifests/kube-apiserver.yaml
	• kube-apiserver -h | grep enable-admission-plugins
From <https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#namespaceautoprovision> 

Two types of admission controllers
	• Validating
	• Mutating

What is the flow of invocation of admission controllers?
Mutating then validating


## API
Identify preferred version:
```
kubectl proxy 8001&
curl localhost:8001/apis/authorization.k8s.io
```

Enable API
Edit api yaml 
Add: 
--runtime-config=rbac.authorization.k8s.io/v1alpha1

APIs
curl http://localhost:6443 -k 
--key admin.key
--cert admin.crt
--cacert ca.crt

curl http://localhost:6443/apis -k | grep name

Check if API is deprecated:
kubectl get --raw /apis/networking.k8s.io/v1beta1/ingresses > out

Kubectl
kube-apiserver -h | grep enable-admission-plugins
kubectl exec kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | grep enable-admission-plugins
Within the kubeapi config/yaml
enable-admission-plugins

### Convert yaml old to new api 
```
kubectl-convert -f ingress-old.yaml --output-version networking.k8s.io/v1 > ingress-new.yaml
kubectl convert -f nginx.yaml --output-version apps/v1
```

## Get Pods
Kubectl get po
	--server
	--client-key
	--client-certificate
	--certificate-authority


## Pattern
Ambassador 
Sidecar
Adapter

## Create Service
```
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
kubectl create service nodeport webapp-service --node-port=30080 --tcp=8080:8080 --dry-run=client -o yaml
kubectl create service nodeport hr-web-app-service --tcp=8080 --node-port=30082 -o yaml --dry-run=client | kubectl set selector --local -f - 'app=hr-web-app' -o yaml | kubectl create -f -
```

## Deployment 
```
kubectl create deploy my-deploy --image=busybox:1.29.2
kubectl set image deploy my-deploy busybox=busybox:1.29.3 --record
kubectl rollout status deploy my-deploy
kubectl rollout history deploy my-deploy 
kubectl rollout history deploy my-deploy --revision=2
kubectl rollout undo deploy my-deploy
```

## How to create a DeamonSet
Create a deployment 
Remove replicas and strategy
Change deployment to DeamonSet 

## Commmand and args 
```
	apiVersion: v1 
	kind: Pod 
	metadata:
	  name: webapp-green
	  labels:
	      name: webapp-green 
	spec:
	  containers:
	  - name: simple-webapp
	    image: kodekloud/webapp-color
	    command: ["python", "app.py"]
	    args: ["--color", "pink"]
```

command: OVERWRITE the container entrypoint 
args: overwrite the pod cmd
	

Set Env Variable in the pod
```
	spec:
	  containers:
	  - env:
	    - name: APP_COLOR
	      value: pink
```

## Security Context
```
kubectl explain pods.spec.containers.securityContext
```
How to check the user running a POD
kubectl exec <pod_name> -- whoami

Where to add Security context 
```
	spec: 
	  securityContext: 
	       runAsUser: 1010
	 containers: 
```
### Security context in the container
```
	apiVersion: v1
	kind: Pod
	metadata:
	  name: ubuntu-sleeper
	  namespace: default
	spec:
	  containers:
	  - command:
	    - sleep
	    - "4800"
	    image: ubuntu
	    name: ubuntu-sleeper
	    securityContext:
	      capabilities:
	        add: ["SYS_TIME"]
```

## Test permission
```
kubectl auth can-i create cm --as system:serviceaccount:project-hamster:processor -n project-hamster
```

## PodAntiAffinity

## TopologySpreadConstraints 

### Ingress
```
kubectl create ingress ingress-test --rule="wear.my-online-store.com/wear*=wear-service:80"
```