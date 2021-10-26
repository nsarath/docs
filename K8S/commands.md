# common

kubectl get pod -n openshift-kube-controller-manager -o jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}' | grep kube-controller-manager-ip | xargs -I% kubectl exec -n openshift-kube-controller-manager % bash -- -c 'kill 1'
    Note: To trigger the garbage collection kill the pid 1 process inside all kube-controller-manager containers:


