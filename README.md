# k8s
## Task 1: Write a Deployment yaml and Apply it

1. Create a `dep-nginx.yaml` using the content given below:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-dep
  labels:
    app: nginx-dep
spec:
  replicas: 3
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
      - name: nginx-ctr
        image: nginx:1.12.2
        ports:
        - containerPort: 80
