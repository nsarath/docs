## Profile-5

```buildoutcfg

cat <<EOF > AIO_PROFILE_5-0000001.yaml
apiVersion: v1
kind: Namespace
metadata:
  labels:
    core.juniper.net/isolated-namespace: 'true'
    name: multi-ns-lb-profile-frontend--0000001
    vn: frontend--0000001-pod
  name: multi-ns-lb-profile-frontend--0000001
---
apiVersion: v1
kind: Namespace
metadata:
  labels:
    core.juniper.net/isolated-namespace: 'true'
    name: multi-ns-lb-profile-middleware--0000001
    vn: middleware--0000001-pod
  name: multi-ns-lb-profile-middleware--0000001
---
apiVersion: v1
kind: Namespace
metadata:
  labels:
    core.juniper.net/isolated-namespace: 'true'
    name: multi-ns-lb-profile-backend--0000001
    vn: backend--0000001-service
  name: multi-ns-lb-profile-backend--0000001
---
apiVersion: core.contrail.juniper.net/v1alpha1
kind: VirtualNetworkRouter
metadata:
  labels:
    vnr: frontend--0000001-pod
  name: frontend--0000001-pod
  namespace: multi-ns-lb-profile-frontend--0000001
spec:
  import:
    virtualNetworkRouters:
    - namespaceSelector:
        matchLabels:
          name: multi-ns-lb-profile-middleware--0000001
      virtualNetworkRouterSelector:
        matchLabels:
          vnr: middleware--0000001-service
  type: mesh
  virtualNetworkSelector:
    matchLabels:
      vn: frontend--0000001-pod
---
apiVersion: core.contrail.juniper.net/v1alpha1
kind: VirtualNetworkRouter
metadata:
  labels:
    vnr: middleware--0000001-service
  name: middleware--0000001-service
  namespace: multi-ns-lb-profile-middleware--0000001
spec:
  import:
    virtualNetworkRouters:
    - namespaceSelector:
        matchLabels:
          name: multi-ns-lb-profile-frontend--0000001
      virtualNetworkRouterSelector:
        matchLabels:
          vnr: frontend--0000001-pod
  type: mesh
  virtualNetworkSelector:
    matchLabels:
      vn: middleware--0000001-service
---
apiVersion: core.contrail.juniper.net/v1alpha1
kind: VirtualNetworkRouter
metadata:
  labels:
    vnr: middleware--0000001-pod
  name: middleware--0000001-pod
  namespace: multi-ns-lb-profile-middleware--0000001
spec:
  import:
    virtualNetworkRouters:
    - namespaceSelector:
        matchLabels:
          name: multi-ns-lb-profile-backend--0000001
      virtualNetworkRouterSelector:
        matchLabels:
          vnr: backend--0000001-service
  type: mesh
  virtualNetworkSelector:
    matchLabels:
      vn: middleware--0000001-pod
---
apiVersion: core.contrail.juniper.net/v1alpha1
kind: VirtualNetworkRouter
metadata:
  labels:
    vnr: backend--0000001-service
  name: backend--0000001-service
  namespace: multi-ns-lb-profile-backend--0000001
spec:
  import:
    virtualNetworkRouters:
    - namespaceSelector:
        matchLabels:
          name: multi-ns-lb-profile-middleware--0000001
      virtualNetworkRouterSelector:
        matchLabels:
          vnr: middleware--0000001-pod
  type: mesh
  virtualNetworkSelector:
    matchLabels:
      vn: backend--0000001-service
---
apiVersion: k8s.cni.cncf.io/v1
kind: NetworkAttachmentDefinition
metadata:
  annotations:
    juniper.net/networks: '{"routeTargetList": null, "importRouteTargetList": null,
      "exportRouteTargetList": null, "ipamV4Subnet": "19.68.66.128/26"}

      '
  name: aap--0000001
  namespace: multi-ns-lb-profile-middleware--0000001
spec:
  config: '{"cniVersion": null, "name": "aap--0000001", "type": "contrail-k8s-cni"}

    '
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend--0000001
  namespace: multi-ns-lb-profile-frontend--0000001
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend--0000001
  template:
    metadata:
      labels:
        app: frontend--0000001
    spec:
      containers:
      - image: svl-artifactory.juniper.net/contrail-nightly/ubuntu-traffic:latest
        name: frontend--0000001
        securityContext:
          capabilities:
            add:
            - NET_ADMIN
          privileged: true
---
apiVersion: v1
kind: Service
metadata:
  name: frontend--0000001
  namespace: multi-ns-lb-profile-frontend--0000001
spec:
  ports:
  - name: port-443
    port: 443
    protocol: TCP
    targetPort: 443
  - name: port-6443
    port: 7878
    protocol: TCP
    targetPort: 7878
  selector:
    app: frontend--0000001
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: middleware--0000001
  namespace: multi-ns-lb-profile-middleware--0000001
spec:
  replicas: 2
  selector:
    matchLabels:
      app: middleware--0000001
  template:
    metadata:
      annotations:
        k8s.v1.cni.cncf.io/networks: '[{"name": "aap--0000001", "cni-args": {"net.juniper.contrail.allowedAddressPairs":
          "[{\"ip\": \"19.68.66.190/32\", \"addressMode\": \"active-standby\"}]"}}]

          '
      labels:
        app: middleware--0000001
    spec:
      containers:
      - image: svl-artifactory.juniper.net/contrail-nightly/ubuntu-traffic:latest
        name: middleware--0000001
        securityContext:
          capabilities:
            add:
            - NET_ADMIN
          privileged: true
---
apiVersion: v1
kind: Service
metadata:
  name: middleware--0000001
  namespace: multi-ns-lb-profile-middleware--0000001
spec:
  ports:
  - name: port-80
    port: 7878
    protocol: TCP
    targetPort: 7878
  selector:
    app: middleware--0000001
  type: ClusterIP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend--0000001
  namespace: multi-ns-lb-profile-backend--0000001
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend--0000001
  template:
    metadata:
      labels:
        app: backend--0000001
    spec:
      containers:
      - image: svl-artifactory.juniper.net/contrail-nightly/ubuntu-traffic:latest
        name: backend--0000001
        securityContext:
          capabilities:
            add:
            - NET_ADMIN
          privileged: true
---
apiVersion: v1
kind: Service
metadata:
  name: backend--0000001
  namespace: multi-ns-lb-profile-backend--0000001
spec:
  ports:
  - name: port-3306
    port: 7878
    protocol: UDP
    targetPort: 7878
  selector:
    app: backend--0000001
  type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend--0000001
  namespace: multi-ns-lb-profile-frontend--0000001
spec:
  egress:
  - ports:
    - port: 53
      protocol: UDP
    - port: 53
      protocol: TCP
  - ports:
    - port: 7878
      protocol: TCP
    #to:
    #- ipBlock:
    #    cidr: 172.30.176.27/32
  ingress:
  - ports:
    - port: 443
      protocol: TCP
    - port: 7878
      protocol: TCP
  podSelector:
    matchLabels:
      app: frontend--0000001
  policyTypes:
  - Ingress
  - Egress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: middleware--0000001
  namespace: multi-ns-lb-profile-middleware--0000001
spec:
  egress:
  - ports:
    - port: 7878
      protocol: UDP
    #to:
    #- ipBlock:
    #    cidr: 172.30.129.125/32
  - ports:
    - port: 53
      protocol: UDP
  - to:
    - ipBlock:
        cidr: 224.0.0.18/32
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: multi-ns-lb-profile-frontend--0000001
      podSelector:
        matchLabels:
          app: frontend--0000001
    ports:
    - port: 80
      protocol: TCP
  - ports:
    - port: 7878
  - from:
    - ipBlock:
        cidr: 224.0.0.18/32
  podSelector:
    matchLabels:
      app: middleware--0000001
  policyTypes:
  - Ingress
  - Egress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend--0000001
  namespace: multi-ns-lb-profile-backend--0000001
spec:
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: multi-ns-lb-profile-middleware--0000001
      podSelector:
        matchLabels:
          app: middleware--0000001
    ports:
    - port: 7878
      protocol: UDP
  podSelector:
    matchLabels:
      app: backend--0000001
  policyTypes:
  - Ingress
---
apiVersion: v1
kind: Pod
metadata:
  annotations:
    k8s.v1.cni.cncf.io/networks: 'aap--0000001

      '
  name: aap-client--0000001
  namespace: multi-ns-lb-profile-middleware--0000001
spec:
  containers:
  - image: svl-artifactory.juniper.net/contrail-nightly/ubuntu-traffic:latest
    name: aap-client--0000001
    securityContext:
      capabilities:
        add:
        - NET_ADMIN
      privileged: true
EOF
kubectl create -f AIO_PROFILE_5-0000001.yaml
#
#
#
kubectl get all -o wide -n multi-ns-lb-profile-frontend--0000001
########################
kubectl get all -o wide -n multi-ns-lb-profile-middleware--0000001
########################
kubectl get all -o wide -n multi-ns-lb-profile-backend--0000001
#




```