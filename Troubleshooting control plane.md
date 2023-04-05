## Lab 5:  Troubleshooting control plane failure 
Execute the below script to simulate control plane failure
```
cat <<EOF | sudo tee /home/ubuntu/script
IyEvYmluL3NoCnN1ZG8gc3UKY3AgL2V0Yy9rdWJlcm5ldGVzL21hbmlmZXN0cy9rdWJlLWFwaXNl
cnZlci55YW1sICAvZXRjL2t1YmVybmV0ZXMva3ViZS1hcGlzZXJ2ZXIueWFtbC5ia3AKc2VkIC1p
IHMvJ3BvcnQ6IDY0NDMnLydwb3J0OiA4MDgwJy9nIC9ldGMva3ViZXJuZXRlcy9tYW5pZmVzdHMv
a3ViZS1hcGlzZXJ2ZXIueWFtbApzZWQgLWkgcy9zZWN1cmUtcG9ydD02NDQzL3NlY3VyZS1wb3J0
PTgwODAvZyAvZXRjL2t1YmVybmV0ZXMvbWFuaWZlc3RzL2t1YmUtYXBpc2VydmVyLnlhbWwK
EOF
```
1.	Verify contents of the script using below
```
cat /home/ubuntu/script
```
2.	Execute below script to test control plane failure.
```
curl -ks https://cka-labs.s3.ap-south-1.amazonaws.com/troubleshooting/controlplane.sh | sh
``` 
3.	Check the status of the cluster using below.
```
kubectl get nodes
```
4.	Check the status of the cluster using below command as well.
```
kubectl cluster-info
``` 
5.	Check the status of the kubelet process using below.
```
journalctl -fu kubelet
``` 
The error says that the kubelet process is unable to connect to the master node , i.e not reachable 172.31.28.206:6443.  
6.	Check the kubectl config view to check the port on which kube-apiserver is configured using below.
```
kubectl config view
```
7.	Check the kube-apiserver.yaml manifest in below path to see  if the right ports are used 
``` 
vi /etc/kubernetes/manifests/kube-apiserver.yaml
```
8.	We do see from config view that the apiserver listens on port 6443. Notice the secure port used in the kube-api-server yaml is 8080 instead of 6443. Also, the ports referenced in the health checks (liveliness, readiness, and startup probes) is also incorrectly used as 8080.  
 
9.	Change the port entries from 8080 to 6443 in the kube-apiserver.yaml manifest as the API Server container  is configured to listen on port 6443 by default.  Save changes to the file using :wq! 
 
10.	 Wait for few seconds and check the status of cluster using below
```
kubectl get nodes
kubectl cluster-info
``` 
 
