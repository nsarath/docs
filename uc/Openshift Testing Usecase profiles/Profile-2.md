## Profile-2

```buildoutcfg


cat <<EOF > AIO_SUBNET_VN_NAD_b.yaml
apiVersion: v1
kind: Namespace
metadata:
  annotations:
    #openshift.io/sa.scc.mcs: s0:c31,c0
    #openshift.io/sa.scc.supplemental-groups: 1000930000/10000
    #openshift.io/sa.scc.uid-range: 1000930000/10000
  labels:
    #core.juniper.net/clusterName: contrail-k8s-kubemanager-ocpd-31
    core.juniper.net/isolated-namespace: "true"
    kubernetes.io/metadata.name: isl-lb-profile--000002
    name: isl-lb-profile-profile2-b
  name: isl-lb-profile-profile2-b
spec:
  finalizers:
  - kubernetes
---
apiVersion: core.contrail.juniper.net/v1alpha1
kind: Subnet
metadata:
  annotations:
    core.juniper.net/description: A subnet is a logical subdivision of an IP network.
    core.juniper.net/display-name: Subnet custom-external-profile2-v4
  name: custom-external-profile2-v4
  namespace: isl-lb-profile-profile2-b
spec:
  cidr: 169.215.21.128/26
---
apiVersion: core.contrail.juniper.net/v1alpha1
kind: VirtualNetwork
metadata:
  annotations:
    core.juniper.net/description: VirtualNetwork is a collection of end points (interface
      or ip(s) or MAC(s)) that can communicate with each other by default. It is a
      collection of subnets whose default gateways are connected by an implicit router
    core.juniper.net/display-name: VN custom-external-profile2
  labels:
    service.contrail.juniper.net/externalNetworkSelector: external-network-in-local-ns
  name: custom-external-profile2
  namespace: isl-lb-profile-profile2-b
spec:
  routeTargetList:
  - target:56096:2989
  v4SubnetReference:
    apiVersion: core.contrail.juniper.net/v1alpha1
    kind: Subnet
    name: custom-external-profile2-v4
    namespace: isl-lb-profile-profile2-b
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-profile2
  namespace: isl-lb-profile-profile2-b
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend-profile2
  template:
    metadata:
      labels:
        app: frontend-profile2
    spec:
      containers:
      - image: svl-artifactory.juniper.net/contrail-nightly/ubuntu-traffic:latest
        name: frontend-profile2
        securityContext:
          capabilities:
            add:
            - NET_ADMIN
          privileged: true
---
apiVersion: v1
kind: Service
metadata:
  annotations:
    service.contrail.juniper.net/externalNetworkSelector: custom-external-in-service-namespace
  name: frontend-profile2
  namespace: isl-lb-profile-profile2-b
spec:
  ports:
  - name: port-21
    port: 7878
    protocol: TCP
    targetPort: 7878
  selector:
    app: frontend-profile2
  type: LoadBalancer
---
apiVersion: "k8s.cni.cncf.io/v1"
kind: NetworkAttachmentDefinition
metadata:
  name: aap-profile2
  namespace: isl-lb-profile-profile2-b
  annotations:
    juniper.net/networks: '{
        "ipamV4Subnet": "183.19.175.64/26",
        "ipamV6Subnet": "1831::/64"
      }'
    core.juniper.net/display-name: aap-profile2
    core.juniper.net/description: aap-profile2
spec:
  config: '{
    "cniVersion": "0.3.1",
    "name": "aap-profile2",
    "type": "contrail-k8s-cni"
  }'
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: middleware-profile2
  namespace: isl-lb-profile-profile2-b
spec:
  replicas: 2
  selector:
    matchLabels:
      app: middleware-profile2
  template:
    metadata:
      annotations:
        k8s.v1.cni.cncf.io/networks: '[{"name": "aap-profile2", "cni-args": {"net.juniper.contrail.allowedAddressPairs":
          "[{\"ip\": \"183.19.175.126/32\", \"addressMode\": \"active-standby\"}]"}}]

          '
      labels:
        app: middleware-profile2
    spec:
      containers:
      - image: svl-artifactory.juniper.net/contrail-nightly/ubuntu-traffic:latest
        name: middleware-profile2
        securityContext:
          capabilities:
            add:
            - NET_ADMIN
          privileged: true
EOF
kubectl create -f AIO_SUBNET_VN_NAD_b.yaml
#
kubectl get all -o wide -n isl-lb-profile-profile2-b

```