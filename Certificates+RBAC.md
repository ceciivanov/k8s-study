# Grant access to the cluster to new user

## Generate certificates

- openssl genrsa -out demouser.key 2048
- openssl req -new -key demouser.key -out demouser.csr -subj "/CN=demouser"
- cat user.csr | base64 | tr -d '\n' -- to get the csr details

## sign certificates manually with ca.crt
openssl x509 -req -in demouser.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out demouser.crt -days 10000

## or use CertificateSigningRequest API
**csr.yaml**

'''apiVersion: certificates.k8s.io/v1\
kind: CertificateSigningRequest\
metadata:\
    name: demouser\
spec:\
    groups:\
    - system:authenticated\
    request: <demouser.csr content>\
    signerName: kubernetes.io/kube-apiserver-client\
    usages:\
    - client auth\'''
         
 - kubectl apply -f csr.yaml
 - kubectl certificate approve demouser
 - kubectl get csr demouser -o jsonpath='{.status.certificate}' | base64 -d > demouser.crt ---- get the certificate

## Setting his environment/config

### define cluster
kubectl config set-cluster kubernetes-demo \
 --server=https://$CLUSTER_IP:6443 \
 --certificate-authority=/etc/kubernetes/pki/ca.crt
 --embed-certs=true \
 --kubeconfig=demouser.conf ----> new config file
 
 ### define user credential in config
 kubectl config set-credentials demouser \
 --client-key=demouser.key
 --client-certificate=demouser.crt \
 --embed-certs=true \
 --kubeconfig=demouser.conf
 
 ### define context
 kubectl config set-context demouser@kubernetes-demo \
 --cluster=kubernetes-demo \
 --user=demouser \
 --kubeconfig=demouser.conf
 
### set-context
**kubectl config use-context demouser@kubernetes-demo --kubeconfig=demouser.conf**

Now demouser has a config file and access to the cluster in his own 'virtual cluster' aka environment

## Grant RBAC to user

kubectl create role/clusterrole demo-role --resource=pods,deployments,services --verb=*
kubectl create rolebinding/clusterrolebinding demo-role-bind --role=demo-role --user=demouser

check access
kubectl auth can-i list pods --as=demouser

**
kubectl get pods --kubeconfig=demouser.conf 
OR new user gets his own home directory and in /root/.kube/config is stored its config file

----------------------------------------------------------------------------------------------

# Check certificates
- openssl req  -noout -text -in ./demouser.csr
- openssl x509  -noout -text -in ./demouser.crt
- openssl x509 -noout -text -in demouser.crt | grep -i validity -A5 # to check validity
