# <span style="color:blue"> CN2-5404 : CN2_OCP : Jdeployer to take Vrouter-gateway addressa and use it during provisioning

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

#### If above doesn't confirm SDN gw, then below is the way to edit to add the vrouter gateway
````
kubectl edit vrouter -n contrail contrail-vrouter-masters -o yaml
kubectl edit vrouter -n contrail contrail-vrouter-nodes -o yaml

#spec:
  agent:
    virtualHostInterface:
      gateway: 10.87.96.188
````

#### After the above changes make sure it effects on the running pods by doing below delete pods
````
kubectl get pods  -n contrail --no-headers=true | awk '/masters|nodes/{print $1}'  | xargs  kubectl delete -n contrail pod 
kubectl delete pod -n contrail contrail-vrouter-masters-bddc
kubectl delete pod -n contrail contrail-vrouter-masters-bddc
kubectl delete pod -n contrail contrail-vrouter-masters-bddc
kubectl delete pod -n contrail contrail-vrouter-nodes-bddc
kubectl delete pod -n contrail contrail-vrouter-nodes-bddc
````

#### now it can be easily verified with the initial command given to check whether showing SDN gw for all vrouters

***
# <span style="color:blue"> Contour ingress controller not coming up</span>

This issue happens due to Port conflict "80" which used by 3rd party Contour and also Redhat Openshift built-in ha-proxy
So, when this issue happen, the envoy pods not come up and the way to solve this is to use different port number for 
Contour in it's Yaml as below by changing to port "89"

````
::
::
2887 ---
2888 apiVersion: v1
2889 kind: Service
2890 metadata:
2891   name: envoy
2892   namespace: projectcontour
2893   annotations:
2894     # This annotation puts the AWS ELB into "TCP" mode so that it does not
2895     # do HTTP negotiation for HTTPS connections at the ELB edge.
2896     # The downside of this is the remote IP address of all connections will
2897     # appear to be the internal address of the ELB. See docs/proxy-proto.md
2898     # for information about enabling the PROXY protocol on the ELB to recover
2899     # the original remote IP address.
2900     service.beta.kubernetes.io/aws-load-balancer-backend-protocol: tcp
2901 spec:
2902   #externalTrafficPolicy: Local
2903   ports:
2904   - port: 89                   <<<<<<<<<<<<<<<<<<<
2905     name: http
2906     protocol: TCP
2907     targetPort: 8080
2908   - port: 449
2909     name: https
2910     protocol: TCP
2911     targetPort: 8443
2912   selector:
2913     app: envoy
2914   type: LoadBalancer
2915 ---
::
::
3075         image: svl-artifactory.juniper.net/atom_virtual_docker/envoyproxy/envoy:v1.18.3
3076         imagePullPolicy: IfNotPresent
3077         name: envoy
3078         env:
3079         - name: CONTOUR_NAMESPACE
3080           valueFrom:
3081             fieldRef:
3082               apiVersion: v1
3083               fieldPath: metadata.namespace
3084         - name: ENVOY_POD_NAME
3085           valueFrom:
3086             fieldRef:
3087               apiVersion: v1
3088               fieldPath: metadata.name
3089         ports:
3090         - containerPort: 8080
3091           hostPort: 89             <<<<<<<<<<<<<<<<<<<
3092           name: http
3093           protocol: TCP
3094         - containerPort: 8443
3095           hostPort: 449
3096           name: https
3097           protocol: TCP
3098         readinessProbe:
````




