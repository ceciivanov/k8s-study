# ETCD in Kubernetes, Backup-Restore


# 1. Stacked ETCD -> etcd is running as static-pod in the controlplane

EXPLORE ETCD:
- cat /etc/kubernetes/manifests/etcd.yaml
- ps -aux | grep etcd
- **--advertise-client-urls it's the etcd server's ip**
- **--listen-client-urls are the hosts that etcd listens on for example localhsot(127.0.0.1) and the host's IP(etcd-server)**

## kubeadm clusters

listens on:
127.0.0.1:2379 is localhost : default value of endpoints, not required if etcd is stacked and is only one
$controlplane-ip:2379

- Same host, different port: --endpoints https://127.0.0.1:port
- Remote host: --endpoints https://host-ip:port (running etcd on other host)
- for the endpoint we set the client-url (from --listen-client-urls), and we run the etcdctl from the server that is running etcd 

## Backup
- ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 
                        --cacert=/etc/kubernetes/pki/etcd/ca.crt \
                        --cert=/etc/kubernetes/pki/etcd/server.crt \
                        --key=/etc/kubernetes/pki/etcd/server.key
                        snapshot save /desired/path/etcd-backup.db


## Restore
for restore endpoints and certs are not required because we are not talking to the API
we just specify the new --data-dir and the backup file, and then adjust the etcd service to point to the new dir
(on static pod default --data-dir is /var/lib/etcd)

- ETCDCTL_API=3 etcdctl --data-dir=/var/lib/etcd-backup snapshot restore /desired/path/etcd-backup.db
- stop kubernetes services by moving the yaml files from /etc/kubernetes/manifests
- edit /etc/kubernetes/manifests/etcd.yaml
- volumes:
-   - hostPath:
      path: /var/lib/etcd    # <- change this
- restart services and kubelet

## manually installed etcd
- stop other services (kube-apiserver etc.)
- change the /etc/systemd/system/etcd.service file --data-dir argument
- systemctl restart etcd
    
    
# 2. External ETCD -- for high availability, meaning that the etcd is a remote server

**backup:**
From the remote host on which you want to run the backup, make sure you have the certs for etcd-server and specify the right endpoints with the etcd-server ip

e.x on controlplane which has external etcd-server
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep etcd
take the --etcd-server IP
and the certs for talking to the etcd 
ETCDCTL_API=3 etcdctl --endpoints=https://$etcd-server-ip:2379 --cacert= --cert= --key= ... snapshot save /path/backup.db

**restore:**

ETCDCTL_API=3 etcdctl --data-dir=/new-data-dir snapshot restore /path/backup.db

- either copy the backup.db to the etcd-server and make the restore there specifying the --data-dir and changing the etcd.service
- also note to make the right permissions: chown -R etcd:etcd /var/lib/etcd-new

either restore from the remote host and move the new data-dir made to the etcd-server
make right permissions and change the etcd-service
