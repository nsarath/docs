## Profile-1

```buildoutcfg

cat <<EOF > AIO_PROFILE_1-0000003.yaml
apiVersion: v1
kind: Namespace
metadata:
  labels:
    core.juniper.net/isolated-namespace: 'true'
    name: isl-np-web-profile-w-haproxy-ingress-0000003
  name: isl-np-web-profile-w-haproxy-ingress-0000003
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-0000003
  namespace: isl-np-web-profile-w-haproxy-ingress-0000003
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend-0000003
  template:
    metadata:
      labels:
        app: frontend-0000003
    spec:
      containers:
      - image: svl-artifactory.juniper.net/contrail-nightly/ubuntu-traffic:latest
        livenessProbe:
          httpGet:
            port: 7868
          initialDelaySeconds: 5
          periodSeconds: 5
        name: frontend-0000003
        securityContext:
          capabilities:
            add:
            - NET_ADMIN
          privileged: true
---
apiVersion: v1
kind: Service
metadata:
  name: frontend-0000003
  namespace: isl-np-web-profile-w-haproxy-ingress-0000003
spec:
  ports:
  - name: port-443
    port: 443
    protocol: TCP
    targetPort: 443
  - name: port-80
    port: 7868
    protocol: TCP
    targetPort: 7868
  selector:
    app: frontend-0000003
  type: NodePort
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: middleware-0000003
  namespace: isl-np-web-profile-w-haproxy-ingress-0000003
spec:
  replicas: 2
  selector:
    matchLabels:
      app: middleware-0000003
  template:
    metadata:
      labels:
        app: middleware-0000003
    spec:
      containers:
      - image: svl-artifactory.juniper.net/contrail-nightly/ubuntu-traffic:latest
        livenessProbe:
          exec:
            command:
            - cat
            - /tmp/am_i_alive
          initialDelaySeconds: 5
          periodSeconds: 5
        name: middleware-0000003
        securityContext:
          capabilities:
            add:
            - NET_ADMIN
          privileged: true
---
apiVersion: v1
kind: Service
metadata:
  name: middleware-0000003
  namespace: isl-np-web-profile-w-haproxy-ingress-0000003
spec:
  ports:
  - name: port-80
    port: 7868
    protocol: TCP
    targetPort: 7868
  selector:
    app: middleware-0000003
  type: ClusterIP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-0000003
  namespace: isl-np-web-profile-w-haproxy-ingress-0000003
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend-0000003
  template:
    metadata:
      labels:
        app: backend-0000003
    spec:
      containers:
      - image: svl-artifactory.juniper.net/contrail-nightly/ubuntu-traffic:latest
        name: backend-0000003
        securityContext:
          capabilities:
            add:
            - NET_ADMIN
          privileged: true
---
apiVersion: v1
kind: Service
metadata:
  name: backend-0000003
  namespace: isl-np-web-profile-w-haproxy-ingress-0000003
spec:
  ports:
  - name: port-3306
    port: 7868
    protocol: UDP
    targetPort: 7868
  selector:
    app: backend-0000003
  type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-0000003
  namespace: isl-np-web-profile-w-haproxy-ingress-0000003
spec:
  egress:
  - ports:
    - port: 53
      protocol: UDP
    - port: 53
      protocol: TCP
  - ports:
    - port: 7868
      protocol: TCP
    #to:
    #- ipBlock:
    #    cidr: 172.30.127.14/32
  ingress:
  - ports:
    - port: 443
      protocol: TCP
    - port: 80
      protocol: TCP
  - ports:
    - port: 7868
      protocol: TCP
  podSelector:
    matchLabels:
      app: frontend-0000003
  policyTypes:
  - Ingress
  - Egress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: middleware-0000003
  namespace: isl-np-web-profile-w-haproxy-ingress-0000003
spec:
  egress:
  - ports:
    - port: 7868
      protocol: UDP
    #to:
    #- ipBlock:
    #    cidr: 172.30.11.226/32
  - ports:
    - port: 53
      protocol: UDP
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend-0000003
    ports:
    - port: 7868
      protocol: TCP
  podSelector:
    matchLabels:
      app: middleware-0000003
  policyTypes:
  - Ingress
  - Egress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-0000003
  namespace: isl-np-web-profile-w-haproxy-ingress-0000003
spec:
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: middleware-0000003
    ports:
    - port: 7868
      protocol: UDP
  podSelector:
    matchLabels:
      app: backend-0000003
  policyTypes:
  - Ingress
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    haproxy.org/path-rewrite: /
    kubernetes.io/ingress.class: haproxy-nodeport
    nginx.ingress.kubernetes.io/rewrite-target: /
  name: ingress-0000003
  namespace: isl-np-web-profile-w-haproxy-ingress-0000003
spec:
  rules:
  - http:
      paths:
      - backend:
          service:
            name: frontend-0000003
            port:
              number: 443
        path: /frontend-0000003-443
        pathType: Prefix
      - backend:
          service:
            name: frontend-0000003
            port:
              number: 7868
        path: /frontend-0000003-80
        pathType: Prefix
EOF
kubectl create -f AIO_PROFILE_1-0000003.yaml
#
#
#
kubectl get all -o wide -n haproxy-controller
#
kubectl get all -o wide -n isl-np-web-profile-w-haproxy-ingress-0000003
#
kubectl get ingress -n isl-np-web-profile-w-haproxy-ingress-0000003 ingress-0000003 -o wide
#
#

```