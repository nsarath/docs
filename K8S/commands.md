# common

kubectl get pod -n openshift-kube-controller-manager -o jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}' | grep kube-controller-manager-ip | xargs -I% kubectl exec -n openshift-kube-controller-manager % bash -- -c 'kill 1'
    
```Note: To trigger the garbage collection kill the pid 1 process inside all kube-controller-manager containers:```



``
[root@helper ocp4]# oc get pods -n default --no-headers -o custom-columns=":.metadata.name,:.spec.dnsPolicy,:.status.phase,:.status.podIP"
simple-a    ClusterFirst   Running   10.131.255.103
simple-x1   ClusterFirst   Running   10.131.255.102
``