## Install NGINX Ingress Controller
### To install the NGINX Ingress Controller in a Kubernetes cluster, run the following command:
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.1/deploy/static/provider/cloud/deploy.yaml
```
This command will create a namespace named ingress-nginx, deploy the necessary resources, and configure the NGINX Ingress Controller.
Verify the NGINX Ingress Controller is running:
```
kubectl get pods -n ingress-nginx --watch
```
Wait for the ingress-nginx-controller pod to have the status Running. Press Ctrl+C to exit the watch mode.

Now, the NGINX Ingress Controller is installed and running in your Kubernetes cluster.

To expose the Ingress Controller externally, you can create a NodePort or LoadBalancer service.
### Deploy Sample Applications with NodePort Services
Create a file named ```sample-apps-nodeport.yaml``` with the following content:
```
apiVersion: v1
kind: Namespace
metadata:
  name: sample-apps

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-app-1
  namespace: sample-apps
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sample-app-1
  template:
    metadata:
      labels:
        app: sample-app-1
    spec:
      containers:
      - name: sample-app-1
        image: nginxdemos/hello:plain-text
        ports:
        - containerPort: 80

---

apiVersion: v1
kind: Service
metadata:
  name: sample-app-1
  namespace: sample-apps
spec:
  type: NodePort
  selector:
    app: sample-app-1
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 31080

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-app-2
  namespace: sample-apps
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sample-app-2
  template:
    metadata:
      labels:
        app: sample-app-2
    spec:
      containers:
      - name: sample-app-2
        image: nginxdemos/hello:plain-text
        ports:
        - containerPort: 80

---

apiVersion: v1
kind: Service
metadata:
  name: sample-app-2
  namespace: sample-apps
spec:
  type: NodePort
  selector:
    app: sample-app-2
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 31081
```
Apply the sample applications with NodePort services using kubectl:
```
kubectl apply -f sample-apps-nodeport.yaml
```
Now, both applications have NodePort services. The sample-app-1 service is exposed on port 31080, and the sample-app-2 service is exposed on port 31081.
## Create a Path-Based Ingress Resource
### Create a file named ```path-based-ingress.yaml``` with the following content:
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-based-ingress
  namespace: sample-apps
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - http:
      paths:
      - path: /app1
        pathType: Prefix
        backend:
          service:
            name: sample-app-1
            port:
              number: 80
      - path: /app2
        pathType: Prefix
        backend:
          service:
            name: sample-app-2
            port:
              number: 80
```
Apply the Ingress resource using kubectl:
```
kubectl apply -f path-based-ingress.yaml
```
## Access Sample Applications
Sample App 1: http://Node-IP:31080/app1
Sample App 2: http://Node-IP:31081/app2
