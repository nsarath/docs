## Profile-4

```buildoutcfg

cat <<EOF > AIO_PROFILE_4-0000001.yaml
## Namespace ##
apiVersion: v1
kind: Namespace
metadata:
  annotations:
    core.juniper.net/forwarding-mode: ip-fabric
  labels:
    core.juniper.net/isolated-namespace: 'true'
    name: multi-ns-contour-ingress-profile-backend-0000001
    vn: backend-0000001-service
  name: multi-ns-contour-ingress-profile-backend-0000001
---
apiVersion: v1
kind: Namespace
metadata:
  annotations:
    core.juniper.net/forwarding-mode: ip-fabric
  labels:
    core.juniper.net/isolated-namespace: 'true'
    name: multi-ns-contour-ingress-profile-middleware-0000001
    vn: middleware-0000001-pod
  name: multi-ns-contour-ingress-profile-middleware-0000001
---
apiVersion: v1
kind: Namespace
metadata:
  annotations:
    core.juniper.net/forwarding-mode: ip-fabric
  labels:
    core.juniper.net/isolated-namespace: 'true'
    name: multi-ns-contour-ingress-profile-frontend-0000001
    vn: frontend-0000001-pod
  name: multi-ns-contour-ingress-profile-frontend-0000001
---
## Service and its Workloads ##
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-0000001
  namespace: multi-ns-contour-ingress-profile-frontend-0000001
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend-0000001
  template:
    metadata:
      labels:
        app: frontend-0000001
    spec:
      containers:
      - image: svl-artifactory.juniper.net/contrail-nightly/ubuntu-traffic:latest
        name: frontend-0000001
        securityContext:
          capabilities:
            add:
            - NET_ADMIN
          privileged: true
---
apiVersion: v1
kind: Service
metadata:
  name: frontend-0000001
  namespace: multi-ns-contour-ingress-profile-frontend-0000001
spec:
  ports:
  - name: port-7868
    port: 7868
    protocol: TCP
    targetPort: 7868
  selector:
    app: frontend-0000001
  type: ClusterIP
---
## Client POD
apiVersion: v1
kind: Pod
metadata:
  name: tier-1-client
  namespace: multi-ns-contour-ingress-profile-frontend-0000001
  labels:
    ab: cd
spec:
  nodeName: worker1
  containers:
    - name: pod-client
      image: svl-artifactory.juniper.net/contrail-nightly/ubuntu-traffic:latest
      securityContext:
       capabilities:
        add:
        - SYS_ADMIN
        - NET_ADMIN
        - NET_RAW
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: middleware-0000001
  namespace: multi-ns-contour-ingress-profile-middleware-0000001
spec:
  replicas: 2
  selector:
    matchLabels:
      app: middleware-0000001
  template:
    metadata:
      labels:
        app: middleware-0000001
    spec:
      containers:
      - image: svl-artifactory.juniper.net/contrail-nightly/ubuntu-traffic:latest
        name: middleware-0000001
        securityContext:
          capabilities:
            add:
            - NET_ADMIN
          privileged: true
---
apiVersion: v1
kind: Service
metadata:
  name: middleware-0000001
  namespace: multi-ns-contour-ingress-profile-middleware-0000001
spec:
  ports:
  - name: port-7868
    port: 7868
    protocol: TCP
    targetPort: 7868
  selector:
    app: middleware-0000001
  type: ClusterIP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-0000001
  namespace: multi-ns-contour-ingress-profile-backend-0000001
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend-0000001
  template:
    metadata:
      labels:
        app: backend-0000001
    spec:
      containers:
      - image: svl-artifactory.juniper.net/contrail-nightly/ubuntu-traffic:latest
        name: backend-0000001
        securityContext:
          capabilities:
            add:
            - NET_ADMIN
          privileged: true
---
apiVersion: v1
kind: Service
metadata:
  name: backend-0000001
  namespace: multi-ns-contour-ingress-profile-backend-0000001
spec:
  ports:
  - name: port-7868
    port: 7868
    protocol: TCP
    targetPort: 7868
  selector:
    app: backend-0000001
  type: ClusterIP
---
## VNR ##
apiVersion: core.contrail.juniper.net/v1alpha1
kind: VirtualNetworkRouter
metadata:
  labels:
    vnr: frontend-0000001-pod
  name: frontend-0000001-pod
  namespace: multi-ns-contour-ingress-profile-frontend-0000001
spec:
  import:
    virtualNetworkRouters:
    - namespaceSelector:
        matchLabels:
          name: multi-ns-contour-ingress-profile-middleware-0000001
      virtualNetworkRouterSelector:
        matchLabels:
          vnr: middleware-0000001-service
  type: mesh
  virtualNetworkSelector:
    matchLabels:
      vn: frontend-0000001-pod
---
apiVersion: core.contrail.juniper.net/v1alpha1
kind: VirtualNetworkRouter
metadata:
  labels:
    vnr: middleware-0000001-service
  name: middleware-0000001-service
  namespace: multi-ns-contour-ingress-profile-middleware-0000001
spec:
  import:
    virtualNetworkRouters:
    - namespaceSelector:
        matchLabels:
          name: multi-ns-contour-ingress-profile-frontend-0000001
      virtualNetworkRouterSelector:
        matchLabels:
          vnr: frontend-0000001-pod
  type: mesh
  virtualNetworkSelector:
    matchLabels:
      vn: middleware-0000001-service
---
apiVersion: core.contrail.juniper.net/v1alpha1
kind: VirtualNetworkRouter
metadata:
  labels:
    vnr: middleware-0000001-pod
  name: middleware-0000001-pod
  namespace: multi-ns-contour-ingress-profile-middleware-0000001
spec:
  import:
    virtualNetworkRouters:
    - namespaceSelector:
        matchLabels:
          name: multi-ns-contour-ingress-profile-backend-0000001
      virtualNetworkRouterSelector:
        matchLabels:
          vnr: backend-0000001-service
  type: mesh
  virtualNetworkSelector:
    matchLabels:
      vn: middleware-0000001-pod
---
apiVersion: core.contrail.juniper.net/v1alpha1
kind: VirtualNetworkRouter
metadata:
  labels:
    vnr: backend-0000001-service
  name: backend-0000001-service
  namespace: multi-ns-contour-ingress-profile-backend-0000001
spec:
  import:
    virtualNetworkRouters:
    - namespaceSelector:
        matchLabels:
          name: multi-ns-contour-ingress-profile-middleware-0000001
      virtualNetworkRouterSelector:
        matchLabels:
          vnr: middleware-0000001-pod
  type: mesh
  virtualNetworkSelector:
    matchLabels:
      vn: backend-0000001-service
---
## Ingress ##
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
  name: ingress-0000001
  namespace: multi-ns-contour-ingress-profile-frontend-0000001
spec:
  ingressClassName: contour
  defaultBackend:
    service:
      name: frontend-0000001
      port:
        number: 7868
---
## httpproxy ##
apiVersion: projectcontour.io/v1
kind: HTTPProxy
metadata:
  name: httpproxy-0000001
  namespace: multi-ns-contour-ingress-profile-frontend-0000001
spec:
  virtualhost:
    fqdn: tour.com
  routes:
  - services:
    - name: frontend-0000001
      port: 7868
    conditions:
    - prefix: /india
    pathRewritePolicy:
      replacePrefix:
      - prefix: /india
        replacement: /
EOF
kubectl create -f AIO_PROFILE_4-0000001.yaml
#
#
#
kubectl get all -o wide -n multi-ns-contour-ingress-profile-frontend-0000001
#
kubectl get ingress -n multi-ns-contour-ingress-profile-frontend-0000001
#
#############################
kubectl get all -o wide -n multi-ns-contour-ingress-profile-middleware-0000001
#############################
kubectl get all -o wide -n multi-ns-contour-ingress-profile-backend-0000001
#
#

```