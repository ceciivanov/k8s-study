 
 # create ingress imperative to get the template

 kubectl create ingress <ingress> -n <namespace> --rule="/path=service:port" --dry-run=client -o yaml

 # check rbac
  
 kubectl auth can-i list pods --as=system:serviceaccount:<namespace>:<serviceaccount> -n <namespace> OR --as=user
