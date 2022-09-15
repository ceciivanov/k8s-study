
# Cluster Upgrade - using kubeadm

## Master Node (controlplane)

- drain node if needed:
kubectl drain controlplane --ignore-daemonsets

### upgrade kubeadm version

- sudo apt update
- sudo apt-mark unhold kubeadm
- sudo apt-get update
- sudo apt-get install -y kubeadm=1.25.0-00
- sudo apt-mark hold kubeadm
- kubeadm version
- sudo kubeadm upgrade plan
- sudo kubeadm upgrade apply v1.25.0

### upgrade kubelet, kubectl
apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet=1.25.x-00 kubectl=1.25.x-00 && \
apt-mark hold kubelet kubectl

sudo systemctl daemon-reload

sudo systemctl restart kubelet

kubectl uncordon controlplane

## Worker Nodes

(on master)
kubectl drain worker-node

(on worker)
apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm=1.25.x-00 && \
apt-mark hold kubeadm
sudo kubeadm upgrade node

apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet=1.25.x-00 kubectl=1.25.x-00 && \
apt-mark hold kubelet kubectl

sudo systemctl daemon-reload

sudo systemctl restart kubelet

kubectl uncordon worker-node
