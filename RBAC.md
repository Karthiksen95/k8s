Lab 3: Control access using RBAC

Provide necessary permissions to pod test-01 inside purple namespace so that it is able perform a GET request to Kubernetes API server.
Task 1: Create a new ServiceAccount demo-sa
1. See the available namespaces.
```
kubectl get ns
```
2. Create a new namespace purple and a ServiceAccount for that namespace.
```
kubectl create ns purple
```
```
kubectl create sa -n purple demo-sa 
```
 
3. Verify that the namespace and service account has been created.
```
kubectl get ns
```
Task 2: Create a new Role and RoleBinding 
1. Create new Role and RoleBinding. 
```
kubectl create role demo-role --verb=list --resource=pods -n purple
```
``` 
kubectl create rolebinding demo-rb --role=demo-role --serviceaccount=purple:demo-sa -n purple
```
Task 3: Test whether you can use a GET request to Kubernetes API 
1. Create a pod with default serviceaccount in purple namespace.
```
kubectl run test --image=nginx -n purple
```
2.  exec into the pod created in previous step.
```
kubectl exec -it -n purple test -- /bin/bash 
```      
3. CURL to the Kube-api server to see whether you are able to list the pods running in purple namespace. 
```
curl -v --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" https://kubernetes.default/api/v1/namespaces/purple/pods 
```
4. Exit from the pod.

5. Now create a pod with demo-sa serviceaccount that has the escalated privileges.
```vi pod-sa.yaml```
```
apiVersion: v1
kind: Pod
metadata:
  name: test-01
  namespace: purple
spec:
  serviceAccountName: demo-sa
  containers:
  - name: my-container
    image: nginx
```
```
kubectl apply -f pod-sa.yaml
```
6. Exec into the pod.
```
kubectl exec -it -n purple test-01 -- /bin/bash 
```
 
7. Now CURL to kube-api server and see whether you can list pods.
``` 
curl -v --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" https://kubernetes.default/api/v1/namespaces/purple/pods 
```
8. Exit from the pod.

 
You will see that it will list the pods in the purple namespace that we were earlier not able to list.
 

