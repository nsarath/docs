# Fabric Forwarding

  This contrail usecase helps Overlay workloads(VM/Pods) part of Virtual-network to communicate with Underlay network nodes/resources without using SDN-Gateway(MX). Once enabled on VN, it works by leaking the VN routes into Fabric default routing table. As also advertized through BGP and so traffic works across multiple Vrouter nodes in the cluster. As lookup and forwarding done at Fabric Default routing table, so there is **no** Overlay MPLS tunnels etc and works by simple IP based forwading.
  
  Incase of Contrail/K8S solution, this can also be used as alternative to policies to enable reachability across multiple Custom VN workloads and Isolated Namespace workloads by enabling "Fabric Forwarding" on them.
  
  Below highlights the Fabric forwarding scenario involving Custom VN workloads and its implementations demonstrated with reference to outputs captured from Control Node introspects and Vrouter-agent routing-tables,
  
  ```
  [root@helper ocp4]# oc get pods -o wide
NAME                READY   STATUS    RESTARTS   AGE     IP               NODE                       NOMINATED NODE   READINESS GATES
ubuntu-custom-a-1   1/1     Running   0          3d5h    105.105.105.3    worker2.ocp4.example.com   <none>           <none>
ubuntu-custom-b-2   1/1     Running   0          2d22h   115.115.115.4    worker0.ocp4.example.com   <none>           <none>
[root@helper ocp4]# 

  [root@helper ocp4]# oc get nodes -o wide
NAME                       STATUS   ROLES    AGE     VERSION           INTERNAL-IP    EXTERNAL-IP   OS-IMAGE                                                       KERNEL-VERSION                 CONTAINER-RUNTIME
master0.ocp4.example.com   Ready    master   5d10h   v1.19.0+b00ba52   192.168.7.21   <none>        Red Hat Enterprise Linux CoreOS 46.82.202106181041-0 (Ootpa)   4.18.0-193.56.1.el8_2.x86_64   cri-o://1.19.2-6.rhaos4.6.git686e6d9.el8
master1.ocp4.example.com   Ready    master   5d10h   v1.19.0+b00ba52   192.168.7.22   <none>        Red Hat Enterprise Linux CoreOS 46.82.202106181041-0 (Ootpa)   4.18.0-193.56.1.el8_2.x86_64   cri-o://1.19.2-6.rhaos4.6.git686e6d9.el8
master2.ocp4.example.com   Ready    master   5d10h   v1.19.0+b00ba52   192.168.7.23   <none>        Red Hat Enterprise Linux CoreOS 46.82.202106181041-0 (Ootpa)   4.18.0-193.56.1.el8_2.x86_64   cri-o://1.19.2-6.rhaos4.6.git686e6d9.el8
worker0.ocp4.example.com   Ready    worker   5d7h    v1.19.0+b00ba52   192.168.7.11   <none>        Red Hat Enterprise Linux CoreOS 46.82.202106181041-0 (Ootpa)   4.18.0-193.56.1.el8_2.x86_64   cri-o://1.19.2-6.rhaos4.6.git686e6d9.el8
worker1.ocp4.example.com   Ready    worker   5d7h    v1.19.0+b00ba52   192.168.7.12   <none>        Red Hat Enterprise Linux CoreOS 46.82.202106181041-0 (Ootpa)   4.18.0-193.56.1.el8_2.x86_64   cri-o://1.19.2-6.rhaos4.6.git686e6d9.el8
worker2.ocp4.example.com   Ready    worker   3d7h    v1.19.0+b00ba52   192.168.7.13   <none>        Red Hat Enterprise Linux CoreOS 46.82.202106181041-0 (Ootpa)   4.18.0-193.56.1.el8_2.x86_64   cri-o://1.19.2-6.rhaos4.6.git686e6d9.el8
[root@helper ocp4]# 

  ```
  
### Before enabling "Fabric Forwarding" on VN
  
   In this example, as shown above command outputs of openshift pods/nodes, ubuntu-custom-a-1 launched on customnetworkA virtual-network on worker2 vrouter and ubuntu-custom-b-1 launched on customnetworkB virtual-network on worker0 vrouter. As we know, above launched workloads by default cannot reach Underlay nodes/resources and also as each of the above workloads being part of different VN and they neither reach each other by default as shown below ping tests,
  
  ```
  root@ubuntu-custom-a-1:/# 
root@ubuntu-custom-a-1:/# ping 192.168.7.21
PING 192.168.7.21 (192.168.7.21) 56(84) bytes of data.
^C
--- 192.168.7.21 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3068ms

root@ubuntu-custom-a-1:/# 
root@ubuntu-custom-a-1:/# ping 115.115.115.4
PING 115.115.115.4 (115.115.115.4) 56(84) bytes of data.
^C
--- 115.115.115.4 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3088ms

root@ubuntu-custom-a-1:/# 
  ```
  
### After enabling "Fabric Forwarding" on VN

  Let us enable Fabric Forwarding on VN using Contrail UI.  Below shows example for CustomNetworkA VN and same to be repeated for CustomNetworkB VN.
  
  
  
  
  

  
  
