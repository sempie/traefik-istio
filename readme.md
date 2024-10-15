## Traefik + Istio

Assume you have a running kubernetes cluster

### Install Istio
```shell
helm repo add --force-update istio https://istio-release.storage.googleapis.com/charts
helm install istio-base istio/base -n istio-system --create-namespace --wait
helm install istiod istio/istiod -n istio-system --wait
```

### Apply Istio config
This will create the app and traefik namespaces and enable istio-injection and peer authentication in them. 
```
kubectl apply -f istio.yaml
```

### Set Traefik Public Domain
set env variable with the domain suffix for dashboard and app (replace with your own)
```
export TRAEFIK_DOMAIN=[traefik.example.local]
sed -i '' 's/TRAEFIK_DOMAIN/'$TRAEFIK_DOMAIN'/g' app.yaml readme.md
```

### Deploy Traefik
This will enable the Kubernetes GatewayAPI provider, enable routes from all namespaces, make sure it is configured to perform nativeLB by default, and allow incoming requests to hit Traefik without redirection by the sidecar.
```shell
helm install traefik -n traefik --wait \
  --version v32.1.1 \
  --set providers.kubernetesIngress.enabled=false \
  --set providers.kubernetesGateway.enabled=true \
  --set providers.kubernetesCRD.nativeLBByDefault=true \
  --set="additionalArguments={--providers.kubernetesgateway.nativelbbydefault=true}" \
  --set 'deployment.podAnnotations.traffic\.sidecar\.istio\.io/includeInboundPorts=''''' \
  --set gateway.listeners.web.namespacePolicy="All" \
  --set image.tag=v3.2.0-rc2 \
  traefik/traefik
```

### Install demo app
```
kubectl apply -f app.yaml
```


## Test
```
curl app.TRAEFIK_DOMAIN/api

{"hostname":"app-v1-c9d9b9fff-r7js4","ip":["127.0.0.1","10.244.1.11"],"headers":{"Accept":["*/*"],"Accept-Encoding":["gzip"],"Content-Length":["0"],"User-Agent":["curl/7.64.1"],"X-B3-Parentspanid":["ca829689cbf5abec"],"X-B3-Sampled":["1"],"X-B3-Spanid":["1de0b90c697bfb72"],"X-B3-Traceid":["09abd11209e6eb7fca829689cbf5abec"],"X-Envoy-Attempt-Count":["1"],"X-Envoy-Internal":["true"],"X-Forwarded-Client-Cert":["By=spiffe://cluster.local/ns/app/sa/default;Hash=594209b12379aac764e2fd62ae4008f4b1818abb3847256df83517e5695a90a3;Subject=\"\";URI=spiffe://cluster.local/ns/traefik/sa/traefik-ingress-controller"],"X-Forwarded-For":["10.244.2.1"],"X-Forwarded-Host":["app.varian.traefiklabs.tech"],"X-Forwarded-Port":["80"],"X-Forwarded-Proto":["http"],"X-Forwarded-Server":["traefik-689d84f4d5-484pj"],"X-Real-Ip":["10.244.2.1"],"X-Request-Id":["2a857832-c7b2-9eca-9c86-0772750e928a"]},"url":"/api","host":"app-v1.app.svc.cluster.local:80","method":"GET"}
```
