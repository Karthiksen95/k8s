```
vi init-cont.yaml
```
```
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
  initContainers:
  - name: date-con
    image: busybox
    command: ["bin/sh","-c"]
    args: [ "date >> /work-dir/index.html" ]

    volumeMounts:
    - name: workdir
      mountPath: "/work-dir"
  volumes:
  - name: workdir
    emptyDir: {}
   ```
   3. Create pod
```
  kubectl apply -f init-cont.yaml 
```

4. Check if pod is running
```
   kubectl get pods
```

5. Describe the pod
```
   kubectl describe po httpd-pod
```

6 Check node on which pod is running and retrieve pod IP from  the "IP" field
```
   kubectl get po -o wide
```
