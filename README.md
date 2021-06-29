# k8s-cluster



spin up three node cluster

* 192.168.33.13 master
* 192.168.33.14 worker-1
* 192.168.33.15 worker-2

see the corresponding post from [here](https://baykara.medium.com/setup-own-kubernetes-cluster-via-virtualbox-99a82605bfcc)

* requirements
```
vagrant
virtualbox
```

Fire up vms
``` 
vagrant up
```
To access each machine respectively via 
```
vagrant ssh master
```
in master node

```
1. set root password
2. switch root account
3. kubeadm init --apiserver-advertise-address 192.168.33.13 --pod-network-cidr=10.244.0.0/16
4. remove --port 0 from /etc/kubernetes/manifests/kube-[controller-api| scheduler].yaml
5. join workers to master node
```
for workers
```
vagrant ssh [worker1|worker2]
```

set to the container runtime  as a containerd
```
cat /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: true
```
show help
```
crictl --help
```

for take to the dashboard token
```
kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
```

for access to the dashboard
```
kubectl proxy --address='0.0.0.0' --accept-hosts='^.*$' --port=8001
```
