# Grant access to the cluster to new user

## Generate certificates

- openssl genrsa -out demouser.key 2048
- openssl req -new -key demouser.key -out demouser.csr -subj "/CN=demouser"
- cat user.csr | base64 | tr -d '\n' -- to get the csr details

## sign certificates manually with ca.crt
openssl x509 -req -in demouser.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out demouser.crt -days 10000

## or use CertificateSigningRequest API
**csr.yaml**

apiVersion: certificates.k8s.io/v1

kind: CertificateSigningRequest

metadata:

    name: demouser
    
spec:

    groups:
    
    - system:authenticated
    
    request: <demouser.csr content>
    
    signerName: kubernetes.io/kube-apiserver-client
    
    usages:
    
    - client auth
         
 - kubectl apply -f csr.yaml
 - kubectl certificate approve demouser
 - kubectl get csr demouser -o jsonpath='{.status.certificate}' | base64 -d > demouser.crt ---- get the certificate

## Setting credentials to the kubeconfig

 ### define user credential in current config (.kube/config)
 kubectl config set-credentials demouser \
 --client-key=demouser.key
 --client-certificate=demouser.crt \
 --embed-certs=true
 
 ### define context
 kubectl config set-context demouser@kubernetes-demo \
 --cluster=kubernetes-demo \
 --user=demouser
 
### set-context
kubectl config use-context demouser@kubernetes-demo

## Grant RBAC to user

kubectl create role/clusterrole demo-role --resource=pods,deployments,services --verb=* (* is for all access)
kubectl create rolebinding/clusterrolebinding demo-role-bind --role=demo-role --user=demouser

check access
kubectl auth can-i list pods --as=demouser

----------------------------------------------------------------------------------------------

# Check certificates
- openssl req  -noout -text -in ./demouser.csr
- openssl x509  -noout -text -in ./demouser.crt
- openssl x509 -noout -text -in demouser.crt | grep -i validity -A5 # to check validity

----------------------------------------------------------------------------------------------

# Service Accounts + RBAC
- Role/RoleBinding
- ClusterRole/ClusterRoleBinding

1. Role + RoleBinding (available in single Namespace, applied in single Namespace)
2. ClusterRole + ClusterRoleBinding (available cluster-wide, applied cluster-wide)
3. ClusterRole + RoleBinding (available cluster-wide, applied in single Namespace)
4. Role + ClusterRoleBinding (NOT POSSIBLE: available in single Namespace, applied cluster-wide)


Create a new ClusterRole named deployment-clusterrole, which only allows to create the following resource types:\
Deployment StatefulSet DeamonSet\
Also create a new ServiceAccount named cicd-token in the existing namespace app-team1.\
Bind the new ClusterRole deployment-clusterrole to the new ServiceAccount cicd-token, limited to the namespace app-team1

- kubectl create ns app-team1
- kubectl create sa cicd-token -n app-team1
- kubectl create clusterrole deployment-clusterrole --resource=deployment,statefulset,daemonset --verb=create
- kubectl create rolebinding cicd-bind --clusterrole=deployment-clusterrole --namespace=app-team1 --serviceaccount=app-team1:cicd-token

## check the access
kubectl auth can-i create deployment --as=system:serviceaccount:app-team1:cicd-token --namespace=app-team1
