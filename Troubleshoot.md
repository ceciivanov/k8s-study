# Troubleshoot Nodes-Kube-apiserver

IF kube-apiserver is healthy and kubectl works
- check that all pods in kube-system namespace are running
- check logs of pods


IF kube-apiserver is not responding

REMEMBER: 
## kubelet is running as process in each node(master-worker)
## kube-apiserver and other components are runnings as static pods (check fies in /etc/kubernetes/manifests)

- sudo systemctl status kubelet
- sudo systemctl restart kubelet
- sudo journalctl -u kubelet | grep -i error
- sudo journalctl | grep apiserver
- ps -ef | grep kubelet
- /var/log/kubelet.log
- /var/log/kube-proxy.log
- cat /var/log/syslog | grep kube-apiserver

- kubelet service config is in: **--config=/var/lib/kubelet/config.yaml**

- systemd unit configuration for kubelet
**/etc/systemd/system/kubelet.service/10-kubeadm.conf**

find differences of kubelet in nodes for example master and worker

LOGS to check
- /var/log/pods
- /var/log/containers
- crictl ps + crictl logs (when kubectl not responind check the containers)
- kubelet logs -> /var/sys/syslog or journalctl

* kubectl get componentstatuses
