# k8sNw.net

Setup napervilleweather.net in a kubernetes cluster as deployment named nwnet; expose as
service nwnet; connect to the internet as napervilleweather.net through an HTTPRoute pointing
to the Envoy Gateway. Get realtime weather data from an NFS mount through the PV/PVC named
`nwcom-persistent-storage` — note, this is the **same PV/PVC used by
[k8sNw.com](https://github.com/jkozik/k8sNw.com)**. No separate PV/PVC is needed for this
deployment.

Source image [InstallNW.net](https://github.com/jkozik/InstallNW.net)

## Directory structure

```
k8sNw.net/
├── nwnet-deploy.yml       # Deployment (1 replica, jkozik/nw.net:v1)
├── nwnet-svc.yml          # NodePort service
├── nwnet-httproute.yaml   # HTTPRoute — napervilleweather.net via Envoy Gateway port 30458
├── README.md              # This file
└── old/
    └── nwnet-ingress.yml  # Retired nginx Ingress (kept for reference)
```

## Prerequisites

- `nwcom-persistent-storage` PV/PVC already created (from [k8sNw.com](https://github.com/jkozik/k8sNw.com))
- Envoy Gateway running with `weather-gateway` on NodePort 30458
- NFS server 192.168.100.153 exporting `/home/nfs/weather-stations/nwcom/public_html`

## Deploy

```bash
cd ~/projects/k8sNw.net

# Apply deployment, service, and HTTPRoute
kubectl apply -f nwnet-deploy.yml
kubectl apply -f nwnet-svc.yml
kubectl apply -f nwnet-httproute.yaml
```

Note: do **not** run `kubectl apply -f .` — that would skip the `old/` directory correctly,
but applying selectively makes the intent explicit.

## Verify

```bash
kubectl get deployment,service,pod,httproute,pvc

# Expected:
# deployment.apps/nwnet   1/1
# service/nwnet           NodePort  80:<nodeport>/TCP
# pod/nwnet-<hash>        1/1 Running
# httproute/nwnet-route   napervilleweather.net, www.napervilleweather.net
# pvc/nwcom-persistent-storage  Bound
```

Test via NodePort directly:
```bash
curl http://<node-ip>:<nodeport>/  | head -5
```

Test via Envoy Gateway (matches production path):
```bash
curl -H "Host: napervilleweather.net" http://<node-ip>:30458/ | head -5
# Should return the Naperville weather HTML
```

## Cloudflare tunnel

Point the `napervilleweather.net` and `www.napervilleweather.net` public hostnames in the
Cloudflare Zero Trust tunnel to:
```
http://<node-ip>:30458
```

## Build image / push to Docker Hub

```bash
docker tag jkozik/nw.net jkozik/nw.net:v1
docker push jkozik/nw.net:v1
```

## NFS share (reference)

The NFS export is on 192.168.100.153 (dell3):
```
/home/nfs/weather-stations/nwcom/public_html  192.168.100.0/24(ro,sync,no_root_squash)
```

This is the same export used by nwcom. Both deployments mount it read-only at
`/var/www/html/mount` inside the container.

## Known gap — /weewx path routing

The old nginx Ingress used rewrite annotations to route `napervilleweather.net/weewx` to a
separate weewx service. The current HTTPRoute only routes `/` to nwnet. If you need
`/weewx` path routing on the new cluster, add a second rule to `nwnet-httproute.yaml`:

```yaml
  - matches:
    - path:
        type: PathPrefix
        value: "/weewx"
    backendRefs:
    - name: weewx
      port: 80
```

This requires the weewx deployment and service to be running first.

## Ingress → HTTPRoute migration

Ingress (nginx) has been deprecated on this cluster. Traffic is managed via the Kubernetes
Gateway API implemented by Envoy Gateway. The old ingress yaml is preserved in `old/` for
reference only — do not apply it on the new cluster.
