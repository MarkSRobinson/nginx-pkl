# NGINX Pkl Chart

This is a re-implementation of the nginx helm chart in Pkl. It provides the same functionality as the base chart but via pkl.

# Usage

To apply the chart directly, use this command.

```bash
pkl chart.pkl | kubectl apply -f -
```

# Extend and configure

### Create a PklProject file

```pkl
amends "pkl:Project"

dependencies {
  ["k8s"] {
    uri = "package://pkg.pkl-lang.org/pkl-k8s/k8s@1.0.1"
  }
  ["chart"] = import("../../nginx-pkl/PklProject")
}
```

### Create your own `chart.pkl`

```pkl
amends "@chart/chart.pkl"

total {
  services {
    default {
      metadata {
        labels {
          ["help"] ="I'm stuck in a yaml factory"
        }
      }
    }
  }
}
```

### Apply  

```bash
pkl chart.pkl | kubectl apply -f -
```

### Result

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/version: 1.0.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: nginx
    app.kubernetes.io/instance: nginx-pkl
    help: I'm stuck in a yaml factory
  name: nginx
spec:
  ports:
  - protocol: TCP
    port: 80
    name: http
    targetPort: http
  type: ClusterIP
---
apiVersion: v1
kind: ConfigMap
metadata:
  labels:
    app.kubernetes.io/version: 1.0.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: nginx
    app.kubernetes.io/instance: nginx-pkl
  name: nginx
data:
  default.conf: |2-
      server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;
        server_name _;

        location / {
          root   /usr/share/nginx/html;
          index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
          root   /usr/share/nginx/html;
        }
      }
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/version: 1.0.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: nginx
    app.kubernetes.io/instance: nginx-pkl
  name: nginx
spec:
  template:
    spec:
      containers:
      - image: krewh/hardened-nginx:1.1.3
        imagePullPolicy: IfNotPresent
        livenessProbe:
          periodSeconds: 10
          initialDelaySeconds: 10
          httpGet:
            path: /
            scheme: HTTPS
            port: https
        ports:
        - protocol: TCP
          name: https
          containerPort: 443
        name: nginx
        readinessProbe:
          periodSeconds: 2
          initialDelaySeconds: 5
          httpGet:
            path: /
            scheme: HTTPS
            port: https
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: nginx
      app.kubernetes.io/instance: nginx-pkl
```