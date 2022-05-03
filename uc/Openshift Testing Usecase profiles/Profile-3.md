## Profile-3

```buildoutcfg

cat <<EOF > AIO_PROFILE_3-0000002.yaml
apiVersion: v1
kind: Namespace
metadata:
  labels:
    name: non-isl-nginx-ingress-lb-profile--0000002
  name: non-isl-nginx-ingress-lb-profile--0000002
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend--0000002
  namespace: non-isl-nginx-ingress-lb-profile--0000002
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend--0000002
  template:
    metadata:
      labels:
        app: frontend--0000002
    spec:
      containers:
      - image: svl-artifactory.juniper.net/contrail-nightly/ubuntu-traffic:latest
        #livenessProbe:
        #  httpGet:
        #    port: 7868
        #  initialDelaySeconds: 5
        #  periodSeconds: 5
        name: frontend--0000002
        securityContext:
          capabilities:
            add:
            - NET_ADMIN
          privileged: true
---
apiVersion: v1
kind: Service
metadata:
  name: frontend--0000002
  namespace: non-isl-nginx-ingress-lb-profile--0000002
spec:
  ports:
  - name: port-80
    port: 7868
    protocol: TCP
    targetPort: 7868
  selector:
    app: frontend--0000002
  type: ClusterIP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: middleware--0000002
  namespace: non-isl-nginx-ingress-lb-profile--0000002
spec:
  # actually 2 but for testcase of patching selector change it is 1
  replicas: 2
  selector:
    matchLabels:
      app: middleware--0000002
  template:
    metadata:
      labels:
        app: middleware--0000002
    spec:
      containers:
      - image: svl-artifactory.juniper.net/contrail-nightly/ubuntu-traffic:latest
        name: middleware--0000002
        securityContext:
          capabilities:
            add:
            - NET_ADMIN
          privileged: true
---
apiVersion: v1
kind: Service
metadata:
  name: middleware--0000002
  namespace: non-isl-nginx-ingress-lb-profile--0000002
spec:
  ports:
  - name: port-443
    port: 7868
    protocol: TCP
    targetPort: 7868
  selector:
    app: middleware--0000002
  type: ClusterIP
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: backend--0000002
  name: backend--0000002-0
  namespace: non-isl-nginx-ingress-lb-profile--0000002
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
    name: backend--0000002-0
    securityContext:
      capabilities:
        add:
        - NET_ADMIN
      privileged: true
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: backend--0000002
  name: backend--0000002-1
  namespace: non-isl-nginx-ingress-lb-profile--0000002
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
    name: backend--0000002-1
    securityContext:
      capabilities:
        add:
        - NET_ADMIN
      privileged: true
---
apiVersion: v1
kind: Service
metadata:
  name: backend--0000002
  namespace: non-isl-nginx-ingress-lb-profile--0000002
spec:
  ports:
  - name: port-3306
    port: 7868
    protocol: UDP
    targetPort: 7868
  selector:
    app: backend--0000002
  type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend--0000002
  namespace: non-isl-nginx-ingress-lb-profile--0000002
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
    to:
    - ipBlock:
        cidr: 172.30.224.175/32
  ingress:
  - ports:
    - port: 7868
      protocol: TCP
  - ports:
    - port: 7868
      protocol: TCP
  podSelector:
    matchLabels:
      app: frontend--0000002
  policyTypes:
  - Ingress
  - Egress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: middleware--0000002
  namespace: non-isl-nginx-ingress-lb-profile--0000002
spec:
  egress:
  - ports:
    - port: 7868
      protocol: UDP
    to:
    - ipBlock:
        cidr: 172.30.157.235/32
  - ports:
    - port: 53
      protocol: UDP
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend--0000002
    ports:
    - port: 7868
      protocol: TCP
  podSelector:
    matchLabels:
      app: middleware--0000002
  policyTypes:
  - Ingress
  - Egress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend--0000002
  namespace: non-isl-nginx-ingress-lb-profile--0000002
spec:
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: middleware--0000002
    ports:
    - port: 7868
      protocol: UDP
  podSelector:
    matchLabels:
      app: backend--0000002
  policyTypes:
  - Ingress
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress--0000002
  namespace: non-isl-nginx-ingress-lb-profile--0000002
  annotations:
    nginx.org/rewrites: "serviceName=frontend--0000002 rewrite=/"
    nginx.org/server-snippets: "server_name ~^.*$;"
    #nginx.org/server-snippets: "server_name 7lvkq7g7.com"
spec:
  ingressClassName: nginx
  rules:
  - host: cafe.com
    http:
      paths:
      - backend:
          service:
            name: frontend--0000002
            port:
              number: 7868
        path: /frontend--0000002-80
        pathType: Prefix
EOF
kubectl create -f AIO_PROFILE_3-0000002.yaml
#
#
#
kubectl get all -o wide -n non-isl-nginx-ingress-lb-profile--0000002
#
kubectl get all -o wide -n nginx-ingress
#
kubectl get ingress -o wide -n non-isl-nginx-ingress-lb-profile--0000002
#
kubectl get ingress -n non-isl-nginx-ingress-lb-profile--0000002
#
#


```