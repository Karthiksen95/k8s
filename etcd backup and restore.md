## Lab : Taking backup of an etcd using snapshot and restore. 

This lab demonstrates the backup process for etcd data store.
### Task 1: Creating Deployments in your cluster
1. Check the versions of kubadm, kubectl and kubelet in the cluster by using following commands, make a count of worker and master nodes.
```
kubeadm version
kubectl version
kubectl get nodes
kubectl cluster-info
```
2. Create 2 applications named frontend and app by using following commands
```
kubectl create deployment frontend --replicas=3 --image=httpd
kubectl expose deploy/frontend --port=80 –type NodePort
```
```
kubectl create deployment app --replicas=2 --image=tomcat
kubectl expose deploy/app --port=8080
```
```
kubectl get deploy
kubectl get svc
```
3. See the nodes in wich the pods are running using the following command
```
kubectl get pods -o wide
```
### Task 2: Backup the cluster configuration by taking a snapshot of etcd
You can take the back of Kubernetes cluster by taking the point in time snapshot of etcd key value store, we will use etcdctl utility for back up in this example.

1. As etcd is running as a static pod in this kubeadm cluster, find out which version of etcd container image is used in the manifest file. You can describe the pod or view the yaml itself. Image will be k8s.gcr.io/etcd:3.4.13-0
```
kubectl get pods -n kube-system
```
``` 
kubectl describe pod < etcd pod name > -n kube-system
```
```
kubectl get pod < etcd pod name > -n kube-system -o yaml
```
```
cat /etc/kubernetes/manifests/etcd.yaml
```
2. Download the script to install etcdctl on master node.
```
wget files.cloudthat.training/devops/kubernetes-cka/etcd-backup.sh
```
3. Run the script to setup and configure etcdctl
```
bash etcd-backup.sh
```
4. Check the version of etcdctl
```
etcdctl --version
```
5. Create a directory named backup in present working directory and copy all Kubernetes certificates from public key infrastructure directory to backup directory, now you have the backup of PKI ready in the backup directory
```
mkdir backup
```
```
cp -rf /etc/kubernetes/pki backup/
```
```
ls backup
```
6. Now take a snapshot of the etcd database by giving required endpoint and certificates as below.
```
ETCDCTL_API=3 etcdctl --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot save backup/etcd-snapshot.db
```
To check the status
```
ETCDCTL_API=3 etcdctl --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot status backup/etcd-snapshot.db
```
7. List the contents of backup directory. You will see the snapshot is created, You can push the entire backup directory to object storages like s3 using a cronjob.
```
ls backup/
```
8. Delete application deployments to see whether the desired state is backedup during restore.
```
kubectl delete deploy frontend
kubectl delete deploy app
```
```
kubectl get deploy
```
```
sudo rm -rf /var/lib/etcd
```
```
sudo mv /etc/kubernetes/manifests/etcd.yaml /etc/kubernetes/manifests/etcd.yaml.bak
```
```
sudo mv /etc/kubernetes/manifests/kube-apiserver.yaml /etc/kubernetes/manifests/kube-apiserver.yaml.bak
```
### Task 3: Restore the cluster configuration 
1. Restore etcd as given below. Create an etcd data directory at /var/lib/etcd. Use 
etcdctl snapshot restore command in a temporary docker container on the snapshot. List the data directory to verify whether the backup was successful. 
```
ETCDCTL_API=3 etcdctl snapshot restore backup/etcd-snapshot.db --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --data-dir=/var/lib/etcd2
```
```
sudo mv /etc/kubernetes/manifests/etcd.yaml.bak /etc/kubernetes/manifests/etcd.yaml
```
```
sudo mv /etc/kubernetes/manifests/kube-apiserver.yaml.bak /etc/kubernetes/manifests/kube-apiserver.yaml
```
Edit the ```etcd.yaml``` file with the new hostpath volume location which is ```/var/lib/etcd2``` and save the file.
```
vi /etc/kubernetes/manifests/etcd.yaml
```
2. Verify the deployment pods.
```
kubectl get pods 
```
3. Now cluster is back you can scale the deployments to ensure that cluster is working properly.
```
kubectl get pods -o wide
kubectl scale deploy frontend --replicas=6
kubectl get pods -o wide
```


 

