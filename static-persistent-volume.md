## Lab 3: Persistent Volume in Kubernetes
This lab demonstrates the functionality and application of Persistent Volumes in Kubernetes. 
### Topics
•	Get node label and create custom index.html on node
•	Create a local PersistentVolume
•	Create a PersistentVolumeClaim
•	Create nginx Pod with nodeSelector
### Task 1: Get Node Label and Create Custom Index.html on Node
1.	View nodes and their labels
Make a note of the kubernetes.io/hostname label of one of the nodes
```
kubectl get nodes --show-labels | grep role=node
```
2.	SSH into one of the nodes using its public IP. Make sure it is the same node of which the hostname is noted
```
ssh admin@<node_public_IP>
```
3.	Switch to root and run the following commands. A directory with custom index.html is created for PersistentVolume mount 
```
sudo su
mkdir /pvdir
echo Hello World! > /pvdir/index.html
exit
exit
```
Task 2: Create a Local Persistent Volume
1.	Create a yaml called pv-volume.yaml and enter its contents as given below
```
vi pv-volume.yaml
```
```
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: "/pvdir"
```
2.	Apply the PersistentVolume yaml created in the previous step

$ kubectl apply -f pv-volume.yaml

 

3.	View details of the PersistentVolume

$ kubectl get pv 

 
Task 3: Create a PersistentVolumeClaim
1.	Create a yaml called pv-claim.yaml and enter its contents as given below

$ vi pv-claim.yaml
 
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
 

2.	Apply the PersistentVolumeClaim yaml created in the previous step

$ kubectl apply -f pv-claim.yaml

 

3.	View details of the PersistentVolumeClaim. Ensure its status is Bound

$ kubectl get pvc 
$ kubectl get pv

 
Task 4: Create nginx Pod with NodeSelector
1.	Create a yaml called pv-pod.yaml and enter its contents as given below. NodeSelector value is entered from hostname value noted from Task 1

$ vi pv-pod.yaml

 

kind: Pod
apiVersion: v1
metadata:
  name: pv-pod
spec:
  volumes:
    - name: pv-storage
      persistentVolumeClaim:
       claimName: pv-claim
  containers:
    - name: pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: pv-storage
  nodeSelector:
    kubernetes.io/hostname: <Your Node Hostname>

Note: The “:” has to be used to map and not “=” in the nodeSelector hostname.

 

2.	Apply the Pod yaml created in the previous step

$ kubectl apply -f pv-pod.yaml

 

3.	View Pod details and see that is created on the required node

$ kubectl get pods -o wide

 

4.	Access shell on a container running in your Pod

$ kubectl exec -it pv-pod -- /bin/bash 

 
 
5.	Run the following commands in the container to verify PersistentVolume

# apt-get update
# apt-get install curl -y
# curl localhost


 

6.	Exit the Pod and delete the resources created in this lab.

$ exit
$ kubectl delete -f pv-pod.yaml
$ kubectl delete -f pv-claim.yaml
$ kubectl delete -f pv-volume.yaml

 
 
