## Lab 7: Troubleshooting application failure in pods in frontend namespace.
1. Create a few deployments, services and pods using below command on the master node.
```
curl -s  https://cka-labs.s3.ap-south-1.amazonaws.com/troubleshooting/app-deploy.sh | sh
```
2. Check status of pods created in frontend namespace.
```
kubectl get po -n frontend
``` 
3. Check the logs for the pods in the frontend namespace using below command. Notice the error resolving the hostname test-db.
```
kubectl logs <pod-name> -n frontend
 ```
4. Notice that based on the pod name, the pods are created as part of the deployment. Verify the deployment name in the frontend namespace.
```
kubectl get deploy -n frontend
```
5. Go to the deployment definition and check for the commands used in the container spec.
```
kubectl get deploy test-web -n frontend -o yaml
```
6. As the container is trying to reach to host name test-db, search for test-db named service in the existing cluster. 
```
kubectl get svc
kubectl get svc -n frontend
``` 
7. As no service objects are found in the default and frontend namespaces, check for other non-system namespaces in the existing cluster.
```
kubectl get ns
``` 
8. Notice the backend namespace. Check the list of components in the backend namespace.
```
kubectl get all -n backend
``` 
9. Notice the test-db service in the backend namespace. Check if the endpoints are populated for the test-db service in the backend namespace.
```
kubectl get ep -n backend
``` 
10. The endpoint is resolving to a pod in the backend namespace.  This should be the correct service to be used in the frontend pod.  The service name (host-db) can be resolved using below. 
```
<service-name>. <namespace>
OR
<service-name>.<namespace>.svc.cluster.local
```
Edit the test-web deployment in the frontend namespace to use the correct service name 
```
kubectl edit deploy test-web -n frontend
``` 
Edit  the command in the spec section to use the proper service name with namespace for proper resolution. Save the edited changes using :wq!
 
 
11. Notice the old pods getting terminated and new pods getting created as below.
``` 
kubectl get pods -n frontendk
``` 
12. Verify the logs for the new pods to see the error resolving the host-db has been fixed. 
```
kubectl get logs -n frontend
kubectl logs <pod-name> -n frontend
``` 
 




 


