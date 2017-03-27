## Annotate any existing ingresses

Existing ingresses that don't have an "ingress.class" will be fought over by a second ingress controller, with a constant fight between controllers to own the ingress.

Add an annotation with the right annotation - "gce" if hosted in Google Cloud. The following will generate commands that you can run:
```
INGRESS="$(kubectl get ingress --all-namespaces -o jsonpath='{range .items[*]}{.metadata.namespace}{"\t"}{.metadata.name}{"\t"}{.metadata.annotations}{"\n"}{end}' | grep -v kubernetes.io/ingress.class)"
if [ -n "$INGRESS" ]; then echo $INGRESS | cut -f1,2 | awk '{print "kubectl --namespace="$1" annotate ingress "$2" kubernetes.io/ingress.class=gce"}'; fi
```

## Get a wildcard SSL certificate for the cluster

### Self signed certificate
To generate a self signed certificate for the domain name you wish to assign to the cluster where the nginx ingress controller is deployed:

```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout star.foo.continuouspipe.net.key -out star.foo.continuouspipe.net.crt -subj "/CN=*.foo.continuouspipe.net/O=CP"`
```

### Where to place it:

Base64 the contents of both the certificate and key, e.g.:
```
base64 star.foo.continuouspipe.net.key
base64 star.foo.continuouspipe.net.crt
```

Update the values in `nginx-ingress/ingress-controller/nginx-ingress-controller-wildcard-ssl-certificate.sslcert.yml` with the base64 key and certificate values.

## Deploy the ingress controller

Deploy a default backend to serve if no server is found for a given hostname:

```
kubectl create ns continuouspipe-system
kubectl --namespace=continuouspipe-system create -f nginx-ingress/default-backend/service.yml
kubectl --namespace=continuouspipe-system create -f nginx-ingress/default-backend/deployment.yml
```

Deploy the ingress controller:
```
kubectl --namespace=continuouspipe-system create -f nginx-ingress/ingress-controller/nginx-load-balancer-conf.yml
kubectl --namespace=continuouspipe-system create -f nginx-ingress/ingress-controller/nginx-ingress-controller-wildcard-ssl-certificate.sslcert.yml
kubectl --namespace=continuouspipe-system create -f nginx-ingress/ingress-controller/service.yml
kubectl --namespace=continuouspipe-system create -f nginx-ingress/ingress-controller/gke-ingress.yml
kubectl --namespace=continuouspipe-system create -f nginx-ingress/ingress-controller/deployment.yml
```

## Set up a wildcard DNS entry

The wildcard DNS should point to the ingress exposed for the ingress controller:

```
$ kubectl --namespace=continuouspipe-system get ingress -o yaml
[...]
  status:
    loadBalancer:
      ingress:
      - ip: 123.123.123.123
````

```
123.123.123.123 *.foo.continuouspipe.net
```


## Deploy your app that uses the nginx ingress controller

To deploy the demo SSL app:
```
kubectl create ns feature-add-ingress-manual
kubectl --namespace=feature-add-ingress-manual create -f nginx-ingress/ingress-controller/nginx-ingress-controller-wildcard-ssl-certificate.sslcert.yml
kubectl --namespace=feature-add-ingress-manual create -f nginx-ingress/demo-app-ssl/feature-add-ingress.services.yml
kubectl --namespace=feature-add-ingress-manual create -f nginx-ingress/demo-app-ssl/feature-add-ingress.ingress-if-port-80-only.yml
kubectl --namespace=feature-add-ingress-manual create -f nginx-ingress/demo-app-ssl/feature-add-ingress.deployment.yml
```
