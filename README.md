## overview architecture
![cicd](./images/ci-cd.png)

## advanced architecture
![cicd](./images/cicd-advanced.png)

## motivation

Such as separating code and manifest repository with argocd, giving multidatacenter and multi cluster support, setting up a test environment with rolling blue/green and running a smoketest, then deploying if everything is ok, and having a full automated deployment system, such as undo. to do a study in which I search for topics and add reference links.

tools
1. argocd
2. argo rollouts
3. argo autopilot < r&d cont.> 

full automated pipeline best practices

code > manifest repo trigger with autopilot > change commit hash > select cluster and app with kustomize > setup test for smoketest > deploy / rollback



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

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/
```
1. set root password
2. switch root account
3. kubeadm init --apiserver-advertise-address 192.168.33.13 --pod-network-cidr=10.244.0.0/16
4. kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
5. kubeadm token create --print-join-command
6. join workers to master node
```
for workers
```
vagrant ssh [worker1|worker2]
```

set to the container runtime  as a containerd
create with nano after below;
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
$ kubectl create serviceaccount dashboard-admin-sa

$ kubectl create clusterrolebinding dashboard-admin-sa --clusterrole=cluster-admin --serviceaccount=default:dashboard-admin-sa

$ kubectl get secrets

$ kubectl describe secret $token_name
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

## finalizers on k8s
kubectl get namespace argocd -o json |jq '.spec = {"finalizers":[]}' >temp.json

## how to install argocd ?
```
$ git clone https://github.com/bilalislam/argocd.git
$ cd argocd/argocd-install/
$ snap install helm --classic
$ helm install argocd ./argo-cd --namespace=argocd    --create-namespace -f values-override.yaml

$ kubectl get pods -n argocd -l app.kubernetes.io/name=argocd-server -o name | cut -d'/' -f 2
```

## how to install argocd - 2  ?
```
$ kubectl create namespace argocd
$ kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

```

##  how to install argo rollouts ?
```
$ kubectl create namespace argo-rollouts
$ kubectl apply -n argo-rollouts -f https://raw.githubusercontent.com/argoproj/argo-rollouts/stable/manifests/install.yaml
```

##  how to watch rollouts pods

```
$ kubectl argo rollouts get rollout <ROLLOUT-NAME> --watch
$ kubectl argo rollouts get experiment <EXPERIMENT-NAME> --watch
```

for svc port change
```
$ kubectl edit svc -n argocd argocd-server

```
```
  ports:
  - name: http
    port: 80
    nodePort: 31346
    protocol: TCP
    targetPort: 8080
  - name: https
    port: 443
    nodePort: 31000
    protocol: TCP
    targetPort: 8080
  selector:
    app.kubernetes.io/instance: argocd
    app.kubernetes.io/name: argocd-server
  sessionAffinity: None
  type: NodePort
  ```

open with 31000 port whatever running on node 

```
$ kubectl get all -n argocd
$ kubectl -n argocd get pods
$ kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
$ argocd account update-password
```

## argo cd cli install
```
$ VERSION=$(curl --silent "https://api.github.com/repos/argoproj/argo-cd/releases/latest" | grep '"tag_name"' | sed -E 's/.*"([^"]+)".*/\1/')

$ curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/$VERSION/argocd-linux-amd64

$ chmod +x /usr/local/bin/argocd

```

## export port for external ip
```
$ kubectl port-forward service/guestbook-ui 8000:80
```

## Argoproj - Application Delivery Platform
1. https://githubmemory.com/repo/alexmt/argo-combined-demo


## best practices
1. seperate code repository & manifest repository
2. if you have not scale number as enougth then you can not get true insight with canary release that's why use to blue/green strategy in this cases.

## gitops principles

![container-mindset](./images/container-buildingblocks.png)

![gitops](./images/gitops-principles.png)

## argocd-autopilot 

![00](./images/argocd-boostrapper.png)


```
$ export GIT_TOKEN=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
$ export GIT_REPO=https://github.com/bilalislam/deployment
$ argocd-autopilot repo bootstrap
$ argocd-autopilot project create testing
$ argocd-autopilot app create hello-world --app github.com/argoproj-labs/argocd-autopilot/examples/demo-app/ -p testing --wait-timeout 2m
$ argocd-autopilot app create rollouts-demo --app github.com/bilalislam/rollouts-demo-deployment/  -p testing --wait-timeout 2m

```

git commit to rollouts-demo after argocd root will get this commit and apply to k8s manifest following belows are;


![01](./images/01.png)

![02](./images/02.png)

![03](./images/03.png)

![04](./images/blue-green.png)

## troobleshoting

1. https://gitlab.cern.ch/clange/gitops-argo-cd/-/issues/1
2. https://bleepcoder.com/argo-cd/430715629/unable-to-create-an-app-from-private-repository
3. https://github.com/argoproj/argo-cd/issues/1322#issuecomment
4. https://github.com/argoproj/argo-cd/issues/1322
5. https://www.gitmemory.com/issue/argoproj/argo-cd/1510/487026607
6. https://stackoverflow.com/questions/52369247/namespace-stuck-as-terminating-how-do-i-remove-it
7. https://github.com/Azure/AKS/issues/733
8. https://docs.google.com/document/d/1gxKxaMQzH9nNDWW9mZV5_cS7EO4S-pm1s_u5aMK-PZQ/edit#heading=h.8bs6j9hp0nx

## roadmap

1. before azure pipeline must build to docker file and generate build number
2. and then create to test app in argocd and trigger to the commit from test and apply k8s file under test folder with enable auto promotion 
3. if made a commit from master then build new tag as under prod k8s folder without enable auto promotion
4. finally apply k8s cluster for test and prod
5. best practices seperate code and manifest repository for clean repo and multi datacenter etc.. as a trendyol sellerads team 

## references
1. https://www.youtube.com/watch?v=0WAm0y2vLIo
2. https://www.youtube.com/watch?v=vSnVhJkyJBw
3. https://www.youtube.com/watch?v=TZgLkCFQ2tk
4. https://www.youtube.com/watch?v=Utjni_LbMxc
5. https://github.com/argoproj-labs/argocd-autopilot
6. https://github.com/argoproj/argo-rollouts/blob/master/docs/getting-started.md
7. https://github.com/argoproj/rollouts-demo/blob/master/examples/analysis/rollout-with-analysis.yaml
8. https://github.com/argoproj/rollouts-demo
