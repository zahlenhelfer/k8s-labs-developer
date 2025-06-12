
# 🧪 Übung: Ingress mit einem einfachen Webserver

---

## 🎯 Ziel

* Ein Deployment mit einem Webserver (NGINX)
* Einen Service vom Typ ClusterIP
* Einen Ingress, der HTTP-Zugriffe auf eine Domain (`example.local`) zu diesem Service weiterleitet

---

## 🧰 Voraussetzungen

* Ein Kubernetes-Cluster
* Ein installierter **Ingress-Controller** (z. B. NGINX)
* Domain `example.local` per `/etc/hosts` auf deinen Cluster zeigen

> 👉 Wenn du z. B. Minikube nutzt, installiere den Ingress-Controller:

```bash
minikube addons enable ingress
```

---

## 🧱 1. Webserver Deployment & Service

**nginx-deploy.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
```

```bash
kubectl apply -f nginx-deploy.yaml
```

---

## 🌐 2. Ingress konfigurieren

**nginx-ingress.yaml**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: example.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-svc
            port:
              number: 80
```

```bash
kubectl apply -f nginx-ingress.yaml
```

---

## 🛠️ 3. Testen per `curl` mit Header-Manipulation

`curl -H "Host: example.local" http://<IP>`

---
## 🛠️ 3a. Lokales DNS setzen

Füge zu deiner `/etc/hosts` folgendes hinzu (Root-Rechte nötig):

```text
<INGRESS_IP> example.local
```

> Die `<INGRESS_IP>` bekommst du z. B. so:

```bash
minikube ip
```

Beispiel:

```text
192.168.49.2 example.local
```

---

## 🔍 4. Test im Browser oder per curl

```bash
curl http://example.local
```

✅ Du solltest die NGINX-Willkommensseite sehen.

---

## 🧹 Aufräumen

```bash
kubectl delete -f nginx-ingress.yaml
kubectl delete -f nginx-deploy.yaml
```

---

## 🧠 Was du gelernt hast

* Wie man einen Service über HTTP per Ingress zugänglich macht
* Ingress-Routen basieren auf **Hostnamen** und **Pfaden**
* Ingress funktioniert nur mit einem aktiven Ingress-Controller
