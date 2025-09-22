
This is a scalable implementation of microservices for platform engineering using
Argo CD, Argo Events, and Argo Workflows, transformative Helm Charts, postgres databases, and Nginx.

Infra:
On an M1 Mac
start:

```bash
vagrant up

vagrant ssh operations-control

vagrant@operations-control:~$ mkdir -p $HOME/.kube
vagrant@operations-control:~$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
vagrant@operations-control:~$ sudo chown $(id -u):$(id -g) $HOME/.kube/config

vagrant@operations-control:~$ kubectl get nodes
NAME                 STATUS   ROLES           AGE     VERSION
operations-control   Ready    control-plane   6m4s    v1.31.13
operations-worker    Ready    <none>          4m44s   v1.31.13


# Install Argo CD
# https://argo-cd.readthedocs.io/en/stable/getting_started/
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

```

stop:
```bash
vagrant destroy -f
```