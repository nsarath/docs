## CN2-5404 : CN2_OCP : Jdeployer to take Vrouter-gateway addressa and use it during provisioning

   This issue only applicable if using Multi-interface like Primary and Secondary subnets for Openshift VRRP deployment
   and only seen when using "Lab team" issued subnets for both Primary & Secondary.

   If this issue happens, it impacts the Traffic drops from Public Endpoint and below is the workaround till this fixed
   in Jdeployer
   
#### below to verify whether all the vrouter masters and nodes take the correct SDN gateway in this case 10.87.96.188
````
root@pxeocp:~/runT# kubectl get pods  -n contrail --no-headers=true | awk '/masters|nodes/{print $1}'  | xargs -I% kubectl exec -n contrail % -c contrail-vrouter-agent -- bash -c 'cat /etc/contrail/contrail-vrouter-[mw]*.conf | grep gateway'  
gateway              = 10.87.96.188
gateway              = 10.87.96.188
gateway              = 10.87.96.188
gateway              = 10.87.96.190
gateway              = 10.87.96.190
root@pxeocp:~/runT# 
````

