# k8s deployment
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
  ```      
2. Apply the Deployment yaml created in the previous step:
```kubectl apply -f dep-nginx.yaml```

3. View the objects created by Kubernetes, Deployment and Replica Set:
```
kubectl get deployments
kubectl get rs
```
4. Access one of the Pods and view nginx version:
```
kubectl get pods
kubectl exec -it <pod_name> -- /bin/bash

nginx -v
exit
```
## Task 2: Update the Deployment with a Newer Image

1. Update the nginx image in Pod using the command below:

    ```sh
    kubectl set image deployment/nginx-dep nginx-ctr=nginx:1.11
    ```

2. Describe the deployment and see that the old pods are replaced with newer ones:

    ```sh
    kubectl describe deployments
    ```

3. Access one of the Pods and view nginx version:

    ```sh
    kubectl get pods
    kubectl exec -it <pod_name> -- /bin/bash
    ```

    ```sh
    nginx -v
    exit
    ```

## Task 3: Rollback of Deployment

1. View the history of Deployments:

    ```sh
    kubectl rollout history deployment/nginx-dep
    ```

2. Rollback the Deployment done in the previous task:

    ```sh
    kubectl rollout undo deployment/nginx-dep --to-revision=1
    kubectl get rs
    ```

3. Access one of the Pods and view nginx version:

    ```sh
    kubectl get pods
    kubectl exec -it <pod_name> -- /bin/bash
    ```

    ```sh
    nginx -v
    exit
    ```

## Task 4: Scaling of Deployments

1. View the number of Pod replicas created by the Deployment:

    ```sh
    kubectl get deployments
    kubectl get pods
    ```

2. Scale up the deployment to have 8 Pod replicas:

    ```sh
    kubectl scale deployment nginx-dep --replicas=8
    ```

3. Check the Pods and deployment, and verify that the number of Pod replicas is 8:

    ```sh
    kubectl get deployments
    kubectl get pods
    ```

4. Scale down the deployments to 2 Pod replicas:

    ```sh
    kubectl scale deployment nginx-dep --replicas=2
    ```

5. Check the Pods and deployment, and verify that the number of Pod replicas is down to 2:

    ```sh
    kubectl get deployments
    kubectl get pods
    ```

6. Clean-up the resources created in this lab:

    ```sh
    kubectl delete -f dep-nginx.yaml
    ```

