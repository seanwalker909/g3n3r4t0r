
This is a scalable implementation of microservices for platform engineering using
Argo CD, Argo Events, and Argo Workflows, transformative Helm Charts, postgres databases, and Nginx.

Infra:
On an M1 Mac
start:

```bash
vagrant up

vagrant ssh operations-control

 mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
 sudo chown $(id -u):$(id -g) $HOME/.kube/config

vagrant@operations-control:~$ kubectl get nodes
NAME                 STATUS   ROLES           AGE     VERSION
operations-control   Ready    control-plane   6m4s    v1.31.13
operations-worker    Ready    <none>          4m44s   v1.31.13

#run this ssh-ed onto the VM, to get kubeconfig
sudo cat /etc/kubernetes/admin.conf

exit #exit vagrant ssh

#copy paste config to local machine:
vi ~/.kube/config

# Install Argo CD
# https://argo-cd.readthedocs.io/en/stable/getting_started/

kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

#get argo initial admin pw
kubectl -n argocd get secrets -o yaml

#open UI, can be done from local machine, thanks to kube config copy above
kubectl port-forward po/argocd-server-d9f4b856-7b5wh -n argocd 8080:8080
```

stop:
```bash
vagrant destroy -f
```