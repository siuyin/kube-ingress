# ingress controller

## mandatory command
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/mandatory.yaml
```

## Custom service provider command

minikube:
```
minikube addons enable ingress
```

bare metal, using NodePort:
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/provider/baremetal/service-nodeport.yaml
```

## Verify installation
```
kubectl get pods --all-namespaces -l app=ingress-nginx --watch
```

Detect installed version:
```
POD_NAMESPACE=ingress-nginx
POD_NAME=$(kubectl get pods -n $POD_NAMESPACE -l app=ingress-nginx -o jsonpath={.items[0].metadata.name})
kubectl exec -it $POD_NAME -n $POD_NAMESPACE -- /nginx-ingress-controller --version
```

## Upgrading
Change the image of the deployment 
in the ingress-nginx namespace
```
kubectl get deploy -n ingress-nginx

kubectl edit deploy nginx-ingress-controller -n ingress-nginx
or
kubectl get deploy -n ingress-nginx  -o yaml > some_file.yml
edit, then kubeapply -f some_file.yml
```
```
kubectl describe deploy -n ingress-nginx nginx-ingress-controller
kubectl get configmaps -n ingress-nginx
kubectl get configmaps -n ingress-nginx nginx-configuration -o yaml >some_file.yml
kubectl get configmaps -n ingress-nginx tcp-services -o yaml >some_file.yml
kubectl get configmaps -n ingress-nginx -services -o yaml >some_file.yml
```
## nginx status
```
curl 192.168.99.101:18080/nginx_status
```

# Configuration
1. ConfigMap: global configuration
1. Annotations: specific config for a particular ingress rule.
1. Custom template: for settings like open_file_cache and listen options.

Annotation keys and values can only be strings. Other types, such as boolean or numeric values must be quoted, i.e. "true", "false", "100".

# Default backend
The default backend exposes two URLs:

* /healthz that returns 200
* / that returns 404

try:
```
curl 192.168.99.101/healthz
curl 192.168.99.101/
```

# Testing

## create a http service
```
kubectl run nginx --image=nginx --port=80
kubectl expose deploy nginx
```

## create an ingress
test-ingress.yml
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /testpath
        backend:
          serviceName: nginx
          servicePort: 80
```
test with:
```
curl -kL http://192.168.99.101/testpath
```

foo-bar-ingress.yml:
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - backend:
          serviceName: nginx
          servicePort: 80
  - host: bar.foo.com
    http:
      paths:
      - backend:
          serviceName: nginx
          servicePort: 80
```

test with:
```
curl -kL 192.168.99.101/ -H 'Host: foo.bar.com'
curl -kL 192.168.99.101/ -H 'Host: bar.foo.com'
```
