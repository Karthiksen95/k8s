Lab 6:  Troubleshooting networking services.
1. Create a test nginx pod using below command on the master node.
```
kubectl run nginx-pod --image=nginx:1.19.1 --port=80 --labels=app=nginx-app
``` 
2. Check status of the created pod 
```
kubectl get po
``` 
3. Expose the pod using a ClusterIP service named nginx-svc using below command.
```
kubectl expose pod nginx-pod --name=nginx-svc --port=80 --target-port=80 --type=ClusterIP
``` 
4. Verify the IP address for the nginx-pod and that the service is pointing to the same IP using describe on the nginx-svc.
```
kubectl get pods -o wide
kubectl describe svc nginx-svc
kubectl get ep nginx-svc
``` 
5. Verify that you are able to curl to the nginx-svc by executing curl from inside a test pod below.
```
kubectl run -i --tty curlpod --image=curlimages/curl -- sh
```
```
curl nginx-svc
exit
``` 
6. Alternatively, verify if the service ‘nginx-svc’ is reachable from the cluster by testing nslookup from inside a test busybox pod as below.
```
kubectl run -i --tty busybox128 --image=busybox:1.28 
```
```
nslookup nginx-svc 
nslookup nginx-svc.default.svc.cluster.local
``` 
7. Verify that the coredns pods are running in the kube-system namespace.
```
kubectl get pods -n kube-system
kubectl get pods -A
``` 
8. Retrieve the IP address of kube-api-server pod running in the kube-system namespace. 
```
kubectl get po kube-apiserver-master -n kube-system -o wide
``` 
9. Verify that the Kubernetes service (svc) created by Kubernetes points to the same IP address as of the kube-apiserver pod in the kube-system namespace.
```
kubectl get svc
kubectl get ep
kubectl describe svc kubernetes
``` 
10. Clean up the pods and services using below commands.
```
kubectl delete pod curlpod busybox128 --force
kubectl delete pod nginx-pod
kubectl delete svc nginx-svc
``` 
 
