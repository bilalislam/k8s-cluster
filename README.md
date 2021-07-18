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

apply dashboard yaml
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml
```

for take to the dashboard token 
```
kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
```

for access to the dashboard
```
kubectl proxy --address='0.0.0.0' --accept-hosts='^.*$' --port=8001
```
Note: don't forget to define port forwarding as 8001:8001 on master node in vbmx nw

for argo cd ref:

https://medium.com/devopsturkiye/self-managed-argo-cd-app-of-everything-a226eb100cf0

for config port forwarding
https://mohitgoyal.co/2021/04/30/setup-continuous-deployment-for-application-with-kubernetes-and-argo-cd/

open to argocd :

https://192.168.33.14:31436/login?return_url=https%3A%2F%2F192.168.33.14%3A31436%2Fapplications

for argocd login:
```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

argocd setup:

https://devops.anaisurl.com/Day-34-GitOps-and-ArgoCD-9cc4142f93b54de088fe3e34fd6524b5


## k8s 
https://devops.anaisurl.com/kubernetes

## continuous delivery
https://continuousdelivery.com/principles/

## k8s guide
https://100daysofkubernetes.io/tools/argo.html#example-notes

## k8s guide github
https://github.com/100daysofkubernetes/100DaysOfKubernetes

## what is the gitops ?
https://thenewstack.io/understanding-gitops-the-latest-tools-and-philosophies/