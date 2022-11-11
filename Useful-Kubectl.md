 
 # create ingress imperative to get the template

 kubectl create ingress ingress -n namespace --rule="/path=service:port" --dry-run=client -o yaml

 kubectl create ingress ingress -n namespace --rule="foo.com/path1*=service1:port" --rule="foo.com/path2*=service2:port" --dry-run=client -o yaml -> this 
set's host foo.com and paths, path* sets as pathType: Prefix

 # check rbac
  
 kubectl auth can-i list pods --as=system:serviceaccount:namespace:serviceaccount -n <namespace> OR --as=user
