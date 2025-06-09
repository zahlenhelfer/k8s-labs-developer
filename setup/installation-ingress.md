```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx && helm repo update
helm install nginx-ingress ingress-nginx/ingress-nginx\
  --create-namespace --namespace ingress-controller\
  --version 4.12.2\
  -f values.yaml
```

```yaml
controller:
  hostNetwork: true
  kind: DaemonSet
  ingressClassResource:
    enabled: true
    default: true
  service:
    externalIPs: [<IP>]

```