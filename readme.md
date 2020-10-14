## Traefik + Istio

Assume you have a running kubernetes cluster

### Install istio

```
curl -L https://istio.io/downloadIstio | sh -
./istio-1.7.3/bin/istioctl install --set profile=demo
```

### Install Traefik

```
k apply -f traefik/
```

### Install demo app

```
k apply -f app/
```

## Test

```
curl app.varian.traefiklabs.tech/api

{"hostname":"app-v1-c9d9b9fff-r7js4","ip":["127.0.0.1","10.244.1.11"],"headers":{"Accept":["*/*"],"Accept-Encoding":["gzip"],"Content-Length":["0"],"User-Agent":["curl/7.64.1"],"X-B3-Parentspanid":["ca829689cbf5abec"],"X-B3-Sampled":["1"],"X-B3-Spanid":["1de0b90c697bfb72"],"X-B3-Traceid":["09abd11209e6eb7fca829689cbf5abec"],"X-Envoy-Attempt-Count":["1"],"X-Envoy-Internal":["true"],"X-Forwarded-Client-Cert":["By=spiffe://cluster.local/ns/app/sa/default;Hash=594209b12379aac764e2fd62ae4008f4b1818abb3847256df83517e5695a90a3;Subject=\"\";URI=spiffe://cluster.local/ns/traefik/sa/traefik-ingress-controller"],"X-Forwarded-For":["10.244.2.1"],"X-Forwarded-Host":["app.varian.traefiklabs.tech"],"X-Forwarded-Port":["80"],"X-Forwarded-Proto":["http"],"X-Forwarded-Server":["traefik-689d84f4d5-484pj"],"X-Real-Ip":["10.244.2.1"],"X-Request-Id":["2a857832-c7b2-9eca-9c86-0772750e928a"]},"url":"/api","host":"app-v1.app.svc.cluster.local:80","method":"GET"}
```
