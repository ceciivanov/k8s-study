# Network Policy

In this case each dash is a specific rule, working as OR condition meaning that traffic is allowed if one of these rule valids 

![image](https://user-images.githubusercontent.com/61785341/191751079-ab08925c-cb36-4f88-9169-ae4b715a6114.png)


Unlike the previous, now there is one rule specified with different AND conditions
![image](https://user-images.githubusercontent.com/61785341/191751295-c8dadede-9103-44ef-b86e-900e4b534bab.png)



![image](https://user-images.githubusercontent.com/61785341/191752613-c8ce3264-e3f4-42cb-9caa-9dd11bde23ac.png)

in a network policy we specify

- from (ingress) 
... rules
- to (egress)

AND

ports:
- protocol:
  port:

in the from section we specify the rules for the allowing traffic (namespaceselector, podselector, ips etc.)
in the ports section we specify the ports that is allowing traffic

![image](https://user-images.githubusercontent.com/61785341/201159219-31cc27db-d27d-4719-8c2f-fd196fd40092.png)
