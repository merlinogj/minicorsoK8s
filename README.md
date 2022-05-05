# minicorsoK8s
> si parte con la creazione della VM fino al deployment di un cluster con kubeadm per poi aggiungere storage e networking

Il minimo per avere un cluster kubernetes è composto da tre VM una control-plane node e due Worker node, i worker possono essere dinamicamente aggiunti in ottica di scalabilità orizzontale.

# Preparazione delle VM 
> parte comune a tutte, installazione di un xubuntu desktop(più facile puoi installare XRDP)

## una volta installato il SO, aggiorniamo i pacchetti:

$ sudo apt update & sudo apt upgrade -y

Procediamo con l'installazione e configurazione di docker o un container runtime a noi congeniale:

``$ sudo apt install docker docker-compose ``

``$ sudo usermod -aG docker $USER``

a questo punto è richiesto un restart del sistema operativo od un log-out/log-in per rileggere i gruppi di appartenenza 

``$ sudo reboot ``
 
 >potete verificare se esiste docker tra i gruppi secondari con il comando  ``$ id ``
 
## Su tutti i nodi l'area di SWAP non deve essere attiva 
* altrimenti il kubelet service non parte 

$ free

$ sudo swapoff -a

$ sudo vi /etc/fstab 

$ sudo systemctl mask dev-sda3.swap

## installare kubeadm, kubectl e kubelet

### source:
* 

## creare una SDN interna a kubernetes di tipo Tigera/Calico

sudo kubeadm init --pod-network-cidr=192.168.0.0/16

kubectl create -f https://projectcalico.docs.tigera.io/manifests/tigera-operator.yaml

kubectl create -f https://projectcalico.docs.tigera.io/manifests/custom-resources.yaml

watch kubectl get pods -n calico-system

kubectl get node

docker ps

kubectl get ns

kubectl get pod -n calico-system -o wide

kubectl get pod -n calico-apiserver -o wide

kubectl get svc -n calico-apiserver

### source: 
* 


## Rook Chep cluster storage:
> Ogni nodo deve avere almeno un disco eg. /dev/sdb non utilizzato per altro
$ lsblk

$ sudo apt-get install -y lvm2

$ lvdisplay 

> per far usare LVM da parte Rook.io 
$ sudo modprobe rbd

### Rook Deployment:
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.7.1/cert-manager.yaml

git clone --single-branch --branch v1.9.2 https://github.com/rook/rook.git

cd rook/deploy/examples/

kubectl create -f crds.yaml -f common.yaml -f operator.yaml

kubectl create -f cluster.yaml

kubectl -n rook-ceph get pod -w

kubectl create -f deploy/examples/toolbox.yaml

kubectl -n rook-ceph rollout status deploy/rook-ceph-tools

kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- bash

### Storage Class:
$ kubectl create -f deploy/examples/csi/rbd/storageclass.yaml

> verifica con $ kubectl get pvc

### source:
* https://rook.io/docs/rook/v1.9/quickstart.html
* https://rook.io/docs/rook/v1.9/pre-reqs.html
* https://rook.io/docs/rook/v1.9/ceph-toolbox.html
* https://rook.io/docs/rook/v1.9/ceph-dashboard.html
* https://github.com/rook/rook/blob/master/Documentation/ceph-block.md # creare la storage class

## Reset Kubernetes Cluster node
sudo kubeadm reset

sudo rm -rf /etc/cni/net.d

rm -rf .kube/
* nel caso non funzioni procedere da docker
* $ docker rm -f $( docker ps -af )


## creare un deployment
kubectl create deployment web --image=nginx --dry-run=client -o yaml 

## rendere kubectl utilizzabile tramite TAB
kubectl completion bash >> .bashrc
source .bashrc


## MINIO BETA
### install krew
> krew auita ad installare i plugin allinterno di kubernetes
$ (   set -x; cd "$(mktemp -d)" &&   OS="$(uname | tr '[:upper:]' '[:lower:]')" &&   ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&   KREW="krew-${OS}_${ARCH}" &&   curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&   tar zxvf "${KREW}.tar.gz" &&   ./"${KREW}" install krew; )

### krew auto completion
$ vi .bashrc

$ source .bashrc
  
$ kubectl krew install minio

$ kubectl minio init

$ kubectl create ns tenant1-ns

$ kubectl minio tenant create tenant1 --servers 4 --volumes 16 --capacity 16Ti --namespace tenant1-ns

$ kubectl get all --namespace minio-operator

$ kubectl minio proxy

$ kubectl port-forward service/minio 443:443 --namespace tenant1-ns

$ kubectl minio tenant expand tenant1 --servers 4 --volumes 16 --capacity 32Gi

### remove minio plugin
$ kubectl krew uninstall minio

$ kubectl delete ns tenant1-ns 

$ kubectl delete ns minio-operator 
### source
* https://krew.sigs.k8s.io/docs/user-guide/setup/install/
* https://docs.min.io/minio/k8s/tenant-management/deploy-minio-tenant-using-commandline.html#deploy-minio-tenant-commandline
* https://github.com/minio/operator/blob/v4.0.10/README.md
* https://docs.min.io/minio/k8s/deployment/deploy-minio-operator.html#deploy-operator-kubernetes
* 

## Stateless application
kubectl apply -f https://k8s.io/examples/application/guestbook/redis-leader-deployment.yaml
kubectl get pod
kubectl logs -f deployment/redis-leader
kubectl apply -f https://k8s.io/examples/application/guestbook/redis-leader-service.yaml
kubectl apply -f https://k8s.io/examples/application/guestbook/redis-follower-deployment.yaml
kubectl apply -f https://k8s.io/examples/application/guestbook/redis-follower-service.yaml
kubectl apply -f https://k8s.io/examples/application/guestbook/frontend-deployment.yaml
kubectl apply -f https://k8s.io/examples/application/guestbook/frontend-service.yaml
kubectl port-forward svc/frontend 8080:80

### source:

## Satefull application
* wordpress
* zookeper
* 

### source:
* https://kubernetes.io/docs/tutorials/stateful-application/

## using kustomization file
mkdir wordpress
mv kustomization.yaml wordpress
mv mysql-deployment.yaml wordpress
mv wordpress-deployment.yaml wordpress
cd wordpress/
ll
vi wordpress-deployment.yaml 
vi mysql-deployment.yaml 
vi kustomization.yaml 
kubectl apply -k ./
  399  cp * ..
  400  ll
  401  cd ..
  402  vi mysql-deployment.yaml 
  403  vi wordpress-deployment.yaml 
  404  kubectl delete -k ./
  405  cd wordpress/
  406  kubectl apply -k ./
  
  407  kubectl config current-context 
  408  kubectl get pod -A -o wide
  409  kubectl delete ns tenant1-ns 
  410  kubectl get pod -A -o wide




## esposizione di servizi in Kubernetes
436  cat mysql.yaml 
  437  kubectl apply -f mysql.yaml 
  438  kubectl apply -f wordpress.yaml 
  439  kubectl get service
  440  kubectl expose deployment wordpress --type=NodePort --name=max
  441  kubectl get svc
  442  ping 10.111.121.234
  443  kubectl delete services wordpress
  444  kubectl describe services max 
  445  kubectl cluster-info
  446  nslookup cluster.info
  447  ping cluster.info
  448  kubectl get svc -A
  449  ping max.svc.cluster.info
  450  ip a
  451  clear
  452  kubectl get svc -A
  453  ping max.default.svc.cluster.info
  454  kubectl -n kube-system describe configmap/coredns
  455  ping max.default.svc.cluster.local
  456  kubectl run --rm -it --restart Never --image busybox bbox -- nslookup minio.heptio-ark-server.svc.cluster.local
  457  kubectl --version
  458  kubectl -v
  459  kubectl v
  460  kubectl version

kubectl expose deployment frontend --name=frontend --type=LoadBalancer

## ingress controller NGINX
  495  kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.2.0/deploy/static/provider/cloud/deploy.yaml
  496  kubectl get pods --namespace=ingress-nginx
  497  kubectl wait --namespace ingress-nginx   --for=condition=ready pod   --selector=app.kubernetes.io/component=controller   --timeout=120s
  498  kubectl get pods --namespace=ingress-nginx
  499  kubectl get svc
  500  kubectl delete svc frontend1
  501  kubectl expose deployment frontend 
  502  kubectl delete svc frontend
  503  kubectl expose deployment frontend 
  504  kubectl create ingress frontend --class=nginx --rule=frontend.localdev.me/*=frontend:80
  510  kubectl port-forward --namespace=ingress-nginx service/ingress-nginx-controller 8080:80
  
  ### source:
  * https://kubernetes.github.io/ingress-nginx/deploy/
  * https://kubernetes.io/docs/concepts/services-networking/service/
  * 
