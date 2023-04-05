## INGRESS DEMO ON KOPS CLUSTER


1. Deploy the base components required to setup ingress controller. These include setup of an ingress-nginx namespace with set of ConfigMaps, 
    ServiceAccount, ClusterRole, and the nginx-ingress controller components.

```
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.27.0/deploy/static/mandatory.yaml
```

2. List the deployed components 
```
   kubectl get pods -A
```

3. Expose the ingress controller using a LoadBalancer service 
```
  kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.27.0/deploy/static/provider/cloud-generic.yaml
```
4. Verify that the ingress LB service is created and exposing the ingress controller
```
   kubectl get svc -n ingress-nginx
```
```
   kubectl describe svc -n ingress-nginx
```

5. Create 2 deployment objects and corresponding services which will serve as backend to which traffic will be routed via the Ingress Controller rules


5A - create a deployment and service for a python based http web server
```
vi dep-svc-pyhttp.yaml
```
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: py-http-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: py-http-web
  template:
    metadata:
      labels:
        app: py-http-web
    spec:
      containers:
      - name: python-http-server
        image: python:2.7
        command: ["/bin/bash"]
        args: ["-c", "echo \" Hello from $(hostname)\" > index.html; python -m SimpleHTTPServer 80"]
        ports:
        - name: http
          containerPort: 80

---
kind: Service
apiVersion: v1

metadata:
  name: pyhttp-demo-svc
spec:
  type: NodePort
  selector:
    app: py-http-web
  ports:
    - name: http
      port: 80
      targetPort: 80
      nodePort: 30011
```
```
 kubectl apply -f dep-svc-pyhttp.yaml
```

5A - create a deployment and service for an httpd server
```
vi dep-svc-httpd.yaml
```
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-deploy
spec:
  replicas: 2
  selector:
    matchLabels:
      app: httpd

  template:
    metadata:
      labels:
        app: httpd
    spec:
      containers:
      - name: httpd
        image: httpd
---
kind: Service
apiVersion: v1

metadata:
  name: httpd-demo-svc
spec:
  type: NodePort
  selector:
    app: httpd
  ports:
    - name: http
      port: 80
      targetPort: 80
      nodePort: 30012
```
```
kubectl apply -f dep-svc-httpd.yaml
```

6. Verify the service objects created and exposed via nodePorts 30011 and 30012
```
   kubectl get svc
```
NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes        ClusterIP   100.64.0.1       <none>        443/TCP        118m
httpd-demo-svc    NodePort    100.69.179.161   <none>        80:30012/TCP   106s
pyhttp-demo-svc   NodePort    100.68.126.37    <none>        80:30011/TCP   7m9s


7. Create an ingress object to configure routing rules
```
  vi ingress-rules.yaml
```
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-demo
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /pyweb
        pathType: Prefix	
        backend:
          service:
            name: pyhttp-demo-svc
            port: 
              number: 80
      - path: /httpd
        pathType: Prefix
        backend:
          service:
            name: httpd-demo-svc
            port: 
              number: 80

```
```
 kubectl apply -f ingress-rules.yaml
```

8. Verify the created ingress object
```
    kubectl get ingress
```

 NAME           CLASS    HOSTS   ADDRESS   PORTS   AGE
 ingress-demo   <none>   *                 80      4s


9. Check the routing rules
```
  kubectl describe ingress
```

E.g. OUTPUT

Name:             ingress-demo
Namespace:        default
Address:          a26d2d5a883244ef88c8820d9fe3fb96-1159428835.ap-south-1.elb.amazonaws.com
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
Rules:
  Host        Path  Backends
  ----        ----  --------
  *
              /pyweb   pyhttp-demo-svc:80 (100.96.1.12:80,100.96.2.12:80,100.96.2.13:80)
              /httpd   httpd-demo-svc:80 (100.96.1.13:80,100.96.1.5:80,100.96.2.14:80 + 1 more...)
Annotations:  nginx.ingress.kubernetes.io/rewrite-target: /
Events:
  Type    Reason  Age   From                      Message
  ----    ------  ----  ----                      -------
  Normal  CREATE  63s   nginx-ingress-controller  Ingress default/ingress-demo
  Normal  UPDATE  37s   nginx-ingress-controller  Ingress default/ingress-demo

10. Get the LoadBalancer URL for the ingress controller service
```
    kubectl get svc -n ingress-nginx
```

   NAME            TYPE           CLUSTER-IP       EXTERNAL-IP                                                                PORT(S)                      AGE
ingress-nginx   LoadBalancer   100.67.191.164   a26d2d5a883244ef88c8820d9fe3fb96-1159428835.ap-south-1.elb.amazonaws.com   80:32558/TCP,443:31749/TCP   122m


11. Navigate tot he External-IP URL/ DNS NAME and append the /pyweb and /httpd path based urls to above ELB , in browser, as well as 
     by curl 


E.g. 

    curl http://a26d2d5a883244ef88c8820d9fe3fb96-1159428835.ap-south-1.elb.amazonaws.com/httpd

    curl http://a26d2d5a883244ef88c8820d9fe3fb96-1159428835.ap-south-1.elb.amazonaws.com/pyweb



  Notice that when you refresh the URL for one ending in /pyweb, you can see traffic load-balanced to different pods within the pyweb deployment
