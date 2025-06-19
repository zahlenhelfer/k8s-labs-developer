
## 🧪 Übung: TLS-Zertifikat für Ingress via Secrets

### 🎯 Ziel

* Ein selbstsigniertes TLS-Zertifikat erstellen
* Es als Kubernetes Secret speichern
* Einen Ingress konfigurieren, der dieses TLS-Secret verwendet
* Zugriff auf eine HTTPS-geschützte Website prüfen

---

### 🔧 Voraussetzungen

* Ingress-Controller ist im Cluster aktiv (z. B. NGINX Ingress via Minikube: `minikube addons enable ingress`)
* `minikube tunnel` wird bei LoadBalancer oder TLS benötigt
* DNS-Fallback per `/etc/hosts` oder `curl --resolve`

---

## 🧩 Schritt 1: TLS-Zertifikat lokal erstellen

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt \
  -subj "/CN=example.local/O=example.local"
```

→ Dateien: `tls.crt`, `tls.key`

---

## 🧩 Schritt 2: Kubernetes TLS Secret erstellen

```bash
kubectl create secret tls example-tls-secret \
  --cert=tls.crt \
  --key=tls.key
```

---

## 🧩 Schritt 3: Test-Deployment + Service

### Datei: `tls-demo-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tls-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tls-demo
  template:
    metadata:
      labels:
        app: tls-demo
    spec:
      containers:
      - name: web
        image: hashicorp/http-echo
        args:
        - "-text=Hello over HTTPS"
        ports:
        - containerPort: 5678
---
apiVersion: v1
kind: Service
metadata:
  name: tls-demo
spec:
  selector:
    app: tls-demo
  ports:
  - port: 80
    targetPort: 5678
```

```bash
kubectl apply -f tls-demo-deployment.yaml
```

---

## 🧩 Schritt 4: Ingress mit TLS

### Datei: `tls-demo-ingress.yaml`

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-demo-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  tls:
  - hosts:
    - example.local
    secretName: example-tls-secret
  rules:
  - host: example.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: tls-demo
            port:
              number: 80
```

```bash
kubectl apply -f tls-demo-ingress.yaml
```

---

## 🧩 Schritt 5: DNS-Zugriff testen (lokal simulieren)

Füge in `/etc/hosts` ein:

```
127.0.0.1 example.local
```

→ Oder verwende `curl` direkt:

```bash
curl -k https://example.local --resolve example.local:443:127.0.0.1
```

→ Ausgabe:
`Hello over HTTPS`

---

## 🧹 Aufräumen

```bash
kubectl delete ingress tls-demo-ingress
kubectl delete secret example-tls-secret
kubectl delete -f tls-demo-deployment.yaml
```

---

## 🧠 Wichtiges Wissen

| Punkt                              | Erklärung                                                   |
| ---------------------------------- | ----------------------------------------------------------- |
| `kubectl create secret tls`        | Speichert `tls.crt` und `tls.key` in TLS-kompatiblem Secret |
| `Ingress.spec.tls`                 | Bindet Secret an Hostname                                   |
| `/etc/hosts` oder `curl --resolve` | Ermöglicht lokale Namensauflösung ohne DNS                  |
| `-k` bei `curl`                    | Ignoriert self-signed Zertifikate (nur zu Testzwecken!)     |

