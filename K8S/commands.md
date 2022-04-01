# Common Commands

kubectl get pod -n openshift-kube-controller-manager -o jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}' | grep kube-controller-manager-ip | xargs -I% kubectl exec -n openshift-kube-controller-manager % bash -- -c 'kill 1'
    
```Note: To trigger the garbage collection kill the pid 1 process inside all kube-controller-manager containers:```



### custom-columns
````
[root@helper ocp4]# oc get pods -n default --no-headers -o custom-columns=":.metadata.name,:.spec.dnsPolicy,:.status.phase,:.status.podIP"
simple-a    ClusterFirst   Running   10.131.255.103
simple-x1   ClusterFirst   Running   10.131.255.102
````

### OC system
````
 oc logs -n openshift-apiserver apiserver-7448648f-dd9f6 -c openshift-apiserver
Error from server: Get "https://172.23.57.202:10250/containerLogs/openshift-apiserver/apiserver-7448648f-dd9f6/openshift-apiserver": x509: certificate signed by unknown authority

oc logs -n openshift-apiserver apiserver-7448648f-dd9f6 -c openshift-apiserver --insecure-skip-tls-verify-backend=true

This is a symptom of the failure state of the cluster and the large number of cluster operators in error state. If you pass --insecure-skip-tls-verify-backend=true you can work around these specific issues.
````


## Root/user access
````
sudo -i 
sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config


systemctl restart sshd
useradd user1
passwd user1
 ****<c0ntrail123>


usermod -G wheel user1
exit
````

### virsh/xargs
````
virsh list --all |  awk '$2 && $2 != "Name" {print $2}' | xargs -n1 virsh destroy
virsh list --all |  awk '$2 && $2 != "Name" {print $2}' | xargs -n1 virsh undefine
rm -Rf /var/lib/libvirt/images/*.*
````

## access through ssh-tunnel

### for 8085
ssh -L 34000:127.0.0.1:34001 root@10.87.3.93
ssh -L 34001:127.0.0.1:34002 root@192.168.7.77
ssh -i .ssh/helper_rsa -L 34002:127.0.0.1:8085 core@192.168.7.12



### OCP UI :    ( kubeadmin / password which shown immediately after the installation complete... )
###needs laptop /etc/hosts with below,
````
192.168.7.77 console-openshift-console.apps.ocp4.example.com oauth-openshift.apps.ocp4.example.com
192.168.7.77 prometheus-k8s-openshift-monitoring.apps.ocp4.example.com oauth-openshift.apps.ocp4.example.com
````


###and tunnel ssh to below,
````
ssh -l root 10.87.64.153 -D 1080

**now OC UI should work::  https://console-openshift-console.apps.ocp4.example.com/dashboards

** Firefox browser supports FoxyProxy which to be setup for SOCKS4 / 127.0.0.1 / 1080 
````

### simple default http server
````
Instead of Wget, use CURL with ubuntu Image				
	After POD up,    "sudo apt-get update". And "sudo apt-get install curl"			
curl -s "http://172.30.8.164:80/?[1-100000]"				
time curl -s "http://yourapp.com/page{1, 2}.php?[1-1000]"				>> Stress multiple consecutive/parallel calls and show Performance
apt-get update; apt-get install tcpdump				
python2 -m SimpleHTTPServer 80 &		>> to start the HTTP server		
				
Traffic				
Virtual Ixia				
Ixia. ((. Statefull sessions  many with  One of client/server being Ixia )				
IPERF				
other load tool of arch.doc		
````

### SSD disk and speed disk-access test

cat /sys/block/sda/queue/rotational

dd if=/dev/zero of=/var/test1 bs=2M count=400 oflag=direct

`confirm the correct disk become Primary bootable`
lsblk
