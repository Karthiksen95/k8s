## Lab: Working with DaemonSets

### Task 1: Create a DaemonSet using the provided YAML

# 1. Create a ds-pod.yaml using the content given below:
```vi ds-pod.yaml```
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluent-ds
  labels:
    app: fluent-ds
spec:
  selector:
    matchLabels:
      app: fluentd-app
  template:
    metadata:
      labels:
        app: fluentd-app
    spec:
      containers:
      - name: fluentd-ctr
        image: fluent/fluentd
```

# 2. Apply the yaml definition to create a fluent-ds DaemonSet:
```
kubectl apply -f ds-pod.yaml
```
# 3. Check the available daemonsets in Kubernetes cluster:
```
kubectl get ds fluent-ds
```
# 4. Verify that pods for Fluentd are created, one for each node using DaemonSet:
```
kubectl get pods -o wide
```
# 5. Cleanup the DaemonSet:
```
kubectl delete -f ds-pod.yaml
```
# 6. Verify the pods to find all the Fluentd pods being deleted from each of the nodes:
```
kubectl get pods
```
