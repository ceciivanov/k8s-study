# RBAC Authorization - Roles, ClusterRoles, RoleBinding, ClusterRoleBinding


## Scenario
- Create a new ClusterRole named deployment-clusterrole, which only allows to create the following resource types:
Deployment
StatefulSet
DeamonSet 
Also create a new ServiceAccount named cicd-token in the existing namespace app-team1. Bind the new ClusterRole deployment-clusterrole to the new ServiceAccount cicd-token, limit to the namespace app-team1

- kubectl create ns app-team1
- kubectl create sa cicd-token -n app-team1
- kubectl create clusterrole deployment-clusterrole --resource=deployment,statefulset,daemonset --verb=create
- kubectl create rolebinding cicd-bind --clusterrole=deployment-clusterrole --namespace=app-team1 --serviceaccount=app-team1:cicd-token
- kubectl auth can-i create deployment --as=system:serviceaccount:app-team1:cicd-token --namespace=app-team1

OR
- create the serviceaccount in default namespace
- kubectl create rolebinding cicd-bind --clusterrole=deployment-clusterrole --namespace=app-team1 --serviceaccount:default:cicd-token

### grant all access
--verb=*
--resource=*

