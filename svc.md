Sure, here's an example README.md file for the tasks outlined above:

Kubernetes Services Lab
This repository contains a set of instructions for setting up and testing Kubernetes Services using different types of Services.

Prerequisites
Before you begin, you will need the following:

A running Kubernetes cluster
kubectl CLI installed and configured to access the Kubernetes cluster
Basic knowledge of Kubernetes concepts and YAML syntax
Task 1: Create a Pod
Open a terminal and create a file named httpd-pod.yaml

Copy the following YAML code into the file:

yaml
Copy code
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
  labels:
    env: prod 
    type: front-end
    app: httpd-ws
spec:
  containers:
  - name: httpd-container
    image: httpd
    ports:
       - containerPort: 80
Save the file and apply the Pod definition using the following command:

bash
Copy code
kubectl apply -f httpd-pod.yaml
Verify that the Pod is running using the following command:

bash
Copy code
kubectl get pods
Describe the Pod using the following command:

bash
Copy code
kubectl describe pod httpd-pod
Task 2: Setup ClusterIP Service
Open a terminal and create a file named httpd-svc.yaml

Copy the following YAML code into the file:

yaml
Copy code
apiVersion: v1
kind: Service
metadata:
  name: httpd-svc
spec:
  selector:
    env: prod
    type: front-end
    app: httpd-ws
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
Save the file and apply the Service definition using the following command:

bash
Copy code
kubectl apply -f httpd-svc.yaml
Verify that the Service is running and has populated the endpoints with IP address matching Pod label using the following command:

bash
Copy code
kubectl describe svc httpd-svc
kubectl get ep
Get the External IPs of the machines in the cluster using the following command:

bash
Copy code
kubectl get nodes -owide | awk '{print $7}'
SSH to one of the machines and rerun the command in the previous step to verify connectivity:

bash
Copy code
ssh -t ubuntu@<Node_IP> curl <Cluster_IP>:<Service_Port>
Task 3: Setup NodePort Service
Modify the Service definition created in the previous task to type NodePort by opening the httpd-svc.yaml file and changing the type field to NodePort.

Apply the changes using the following command:

bash
Copy code
kubectl apply -f httpd-svc.yaml
View details of the modified Service using the following command:

bash
Copy code
kubectl describe svc httpd-svc
Validate connectivity using External IP on NodePort using the following command:

bash
Copy code
curl <EXTERNAL-IP>:NodePort
