# AKS-ingress-controller

We'll be using Helm binary for the ingress controller deployment. Hence the first step that we will be needing is downloading Helm binary:

```
1. cd /tmp
2. curl -k -o /tmp/helm3.tar.gz <URL for downloading Helm3>
3. curl -k -o /tmp/nginx-ingress.tgz <URL for downloading nginx ingress controller>
tar -xvzf helm3.tar.gz

```

### helm install nginx ingress controller

```
/tmp/linux-amd64/helm upgrade --install ingress /tmp/nginx-ingress.tgz \
    --namespace <namespace name>
    --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-internal"=true \
    --set controller.image.repository="conainer-registry.ubs.net/kubernetes-ingress-controller/nginx-ingress-controller" \
    --set controller.image.tag="0.26.1" \
    --set controller.resources.requests.cpu="250m" \
    --set controller.resources.requests.memory="64Mi" \
    --set controller.resources.limits.cpu="500m" \
    --set controller.resources.limits.memory="128Mi" \
    --set defaultBackend.image.repository="container-registry.ubs.net/google-containers/defaultbackend" \
    --set defaultBackend.image.tag="1.4" \
    --set defaultBackend.resources.requests.cpu="500m" \
    --set defaultBackend.resources.requests.memory="512Mi" \
    --set defaultBackend.resources.limits.cpu="1000m" \
    --set defaultbackend.resources.limits.memory="1042Mi"
```

### When Deployed, you will get:

```
kubectl getpods
NAME  READY  STATUS  RESTARTS AGE
ingress-nginx-ingress-controller-577d8f6864-r8g2c 1/1 Running 0  87h
ingress-nginx-ingress-default-backend-577f7y6864-t8f2c 1/1 Running 0  87h
```

### You should also have a serice running after few minutes with external IP

```
kubectl get sevc
NAME  TYPE  CLUSTER-IP  EXTERNAL-IP  PORT(S)  AGE
ingress-nginx-ingress-controller  LoadBalancer 10.215.16.138 10.214.40.9 80:32237/TCP  443:30024/TCP 28h
ingress-nginx-ingress-default-backend  ClusterIP 10.251.15.115 <none>  80/TCP 28h
``` 

### Example of application configured to use the ingress controller

apiVersion: apps/v1
kind: Deployment
metadata:
  name: k8s-hello-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: k8s-hello-app
  template:
    metadata:
      labels:
        app: k8s-hello-app
      apppodbinding: whisky
  spec:
    containers:
    - name: k8s-hello-app
      inage: container-registry.ubs.net/bdent/k8s-hello-whisky:1.0.70-SNAPSHOT
      resources:
        limits:
          cpu: "1000m"
          memory: "256Mi"
    ports:
    - containerPort: 8000
---
apiVersion: v1
kind: Service
metadata:
  name: k8s-hello
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 8000
  selector: k8s-hello-app
```
