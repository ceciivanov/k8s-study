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
