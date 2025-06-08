
# 🧪 Übung: TLS-Zertifikate erzeugen & in NGINX per Secret nutzen

---

## 1. Zertifikate lokal erzeugen

```bash
openssl req -x509 -nodes -days 365 \
  -newkey rsa:2048 \
  -keyout tls.key \
  -out tls.crt \
  -subj "/CN=demo.local"
```

---

## 2. Secret in Kubernetes anlegen

```bash
kubectl create secret generic my-tls-secret \
  --from-file=tls.crt=./tls.crt \
  --from-file=tls.key=./tls.key
```

---

## 3. NGINX Deployment mit HTTPS und Secret als Volumes

**nginx-tls.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-tls
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-tls
  template:
    metadata:
      labels:
        app: nginx-tls
    spec:
      containers:
      - name: nginx
        image: nginx:stable-alpine
        ports:
        - containerPort: 443
        volumeMounts:
        - name: tls-secret
          mountPath: /etc/nginx/certs
          readOnly: true
        - name: nginx-config
          mountPath: /etc/nginx/conf.d
      volumes:
      - name: tls-secret
        secret:
          secretName: my-tls-secret
      - name: nginx-config
        configMap:
          name: nginx-tls-config
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-tls-svc
spec:
  type: NodePort
  selector:
    app: nginx-tls
  ports:
  - port: 443
    targetPort: 443
    nodePort: 30443
```

---

## 4. NGINX ConfigMap mit HTTPS Server-Block

**nginx-config.yaml**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-tls-config
data:
  default.conf: |
    server {
        listen 443 ssl;
        server_name demo.local;

        ssl_certificate /etc/nginx/certs/tls.crt;
        ssl_certificate_key /etc/nginx/certs/tls.key;

        location / {
            root /usr/share/nginx/html;
            index index.html;
        }
    }
```

```bash
kubectl apply -f nginx-config.yaml
kubectl apply -f nginx-tls.yaml
```

---

## 5. Zugriff auf den NGINX-Server

```bash
kubectl get svc nginx-tls-svc
```

Zugriff über NodePort z.B.:

```
https://<Node-IP>:30443
```

Da es ein Self-Signed-Zertifikat ist, wirst du wahrscheinlich eine Browser-Warnung bekommen — das ist normal.

---

## 6. Test im Pod (optional)

```bash
kubectl exec -it $(kubectl get pods -l app=nginx-tls -o jsonpath="{.items[0].metadata.name}") -- \
  curl -k https://localhost
```

---

## 7. Aufräumen

```bash
kubectl delete deployment nginx-tls
kubectl delete service nginx-tls-svc
kubectl delete configmap nginx-tls-config
kubectl delete secret my-tls-secret
rm tls.crt tls.key
```

---

# 🧠 Zusammenfassung

* Du hast lokal TLS-Zertifikate erzeugt
* Diese in Kubernetes als Secret gespeichert
* NGINX so konfiguriert, dass es diese Zertifikate für HTTPS nutzt
* NGINX als Deployment + Service mit NodePort exposed
