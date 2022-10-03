## There are currently two different kinds of affinity that you can define:

- Node Affinity – Used to constrain the Nodes that can receive a Pod by matching labels of those Nodes. 
Node Affinity can only be used to set positive affinities that attract Pods to the Node.

- Inter-Pod Affinity – Used to constrain the Nodes that can receive a Pod by matching labels of the existing Pods already running on each of those Nodes. 
Inter-Pod Affinity can be either an attracting affinity or a repelling anti-affinity.


In the simplest possible example, a Pod that includes a Node Affinity condition of label=value will only be scheduled to Nodes with a label=value label. 
A Pod with the same condition but defined as an Inter-Pod Affinity will be scheduled to a Node that already hosts a Pod with a label=value label.


- requiredDuringSchedulingIgnoredDuringExecution – This is the “hard” affinity matcher that requires the Node meet the constraints you define.
- preferredDuringSchedulingIgnoredDuringExecution – This is the “soft” matcher to express a preference that’s ignored when it can’t be fulfilled.


example: Pod will be scheduled to node that has labels, hardware-class with value a,b OR c,  AND internal label exists (label="")

affinity:

    nodeAffinity:
    
        requiredDuringSchedulingIgnoredDuringExecution:
        
            nodeSelectorTerms:
          
            - matchExpressions:
            
                - key: hardware-class
                
                  operator: In
                  
                  values:
               
                  - a
                
                  - b
                
                  - c
          
            - matchExpressions:
            
                - key: internal
            
                  operator: Exists
              
   
   Multiple - matchExpressions - AND condition
   
   Multiple nodeSelectorTerms: - OR condition
