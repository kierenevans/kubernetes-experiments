# Nginx Ingress controller

```
kubectl create namespace ingress-system

kubectl --namespace=ingress-system create -f default-backend/deployment.yml
kubectl --namespace=ingress-system create -f default-backend/service.yml

kubectl --namespace=ingress-system create -f ingress-controller/deployment.yml
kubectl --namespace=ingress-system create -f ingress-controller/service.yml
# Update this file with your own certificate:
kubectl --namespace=ingress-system create -f ingress-controller/ssl-cert.yml
kubectl --namespace=ingress-system create -f ingress-controller/gke-ingress.yml

kubectl create -f demo-app/deployment.yml
kubectl create -f demo-app/ingress.yml
kubectl create -f demo-app/service.yml
```
