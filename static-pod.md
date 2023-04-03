## STATIC POD

1.  Login to any of the worker nodes 


2. Elevate to root and verify kubelet is running
```
  sudo su
  systemctl status kubelet.service
```

3.Open 10-kubeadm.conf from /etc/systemd/system/kubelet.service.d directory and find the path for the kubelet config file and open it
```
  cd /etc/systemd/system/kubelet.service.d
  ls
  vi 10-kubeadm.conf
```
4. Open config.yaml and on line 30, find the path of the static pods 
```
    vi /var/lib/kubelet/config.yaml
```

5. Go to the /etc/kubernetes/manifest location and create a file named pod.yaml with below content
```
   cd /etc/kubernetes/manifests
```
```
   vi pod.yaml
```
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-static-web
  labels:
    name: nginx-static-pod
spec:
  containers:
    - name: nginx-pod
      image: nginx
      ports:
        - name: web-port
          containerPort: 80
          protocol: TCP

```
6.Restart the kubelet service on worker node
```
   sudo systemctl restart kubelet.service
```
7. SWITCH TO MASTER NODE (IN SEPERATE TERMINAL) and check status of pods using kubectl
```
   kubectl get pods
```
8. View the pod on wider scale and notice that pod is being scheduled to the specific worker node (on which kubelet was restarted)
```
   kubectl get pods -o wide
```
9. Try deleting the pod  and verify it comes back after some time
```
   kubectl delete pod <pod-name>
```
```   
   kubectl get pods
```

10. SWITCH TO THE WORKER NODE TERMINAL
    Delete the file  pod.yaml from /etc/kubernetes/manifests on the specific worker node
```
  rm -f /etc/kubernetes/manifests/pod.yaml
```


 
