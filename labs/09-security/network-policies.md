
# 🧪 Übung: Kubernetes NetworkPolicies verstehen & anwenden

---

## 🎯 Ziel der Übung

* Einen Namespace und 3 Pods mit unterschiedlichen Labels erstellen
* Ohne NetworkPolicy: alle können mit allen kommunizieren
* Mit NetworkPolicy: nur bestimmte Pods dürfen ein Ziel erreichen
* Netzwerkzugriff mit `curl` testen

---

## 🧱 Vorbereitung

### 🔧 1. Namespace anlegen

```bash
kubectl create namespace netpol-demo
```

---

### 📦 2. Drei Test-Pods starten

```yaml
# netpol-pods.yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
  namespace: netpol-demo
  labels:
    role: frontend
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "while true; do sleep 3600; done"]

---
apiVersion: v1
kind: Pod
metadata:
  name: backend
  namespace: netpol-demo
  labels:
    role: backend
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "while true; do sleep 3600; done"]

---
apiVersion: v1
kind: Pod
metadata:
  name: attacker
  namespace: netpol-demo
  labels:
    role: attacker
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "while true; do sleep 3600; done"]
```

```bash
kubectl apply -f netpol-pods.yaml
```

---

### 🌐 3. Einfachen Webserver im Backend starten

```bash
kubectl exec -n netpol-demo backend -- sh -c "nohup httpd -f -p 80 &"
```

---

## 🧪 Test 1: Kommunikation ist **frei**

```bash
kubectl exec -n netpol-demo frontend -- wget -qO- http://backend
kubectl exec -n netpol-demo attacker -- wget -qO- http://backend
```

✅ Beide Pods können den `backend` erreichen.

---

## 🛡️ 4. NetworkPolicy: Nur `frontend` darf `backend` erreichen

```yaml
# netpol-allow-frontend.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
  namespace: netpol-demo
spec:
  podSelector:
    matchLabels:
      role: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 80
  policyTypes:
  - Ingress
```

```bash
kubectl apply -f netpol-allow-frontend.yaml
```

---

## 🧪 Test 2: Zugriff prüfen

```bash
# Darf durch (erlaubt in der Policy)
kubectl exec -n netpol-demo frontend -- wget -qO- http://backend

# Darf NICHT durch (nicht erlaubt)
kubectl exec -n netpol-demo attacker -- wget -qO- http://backend
```

✅ Der `frontend` kann weiterhin auf `backend` zugreifen
❌ Der `attacker` erhält einen Timeout

---

## 🧹 Aufräumen

```bash
kubectl delete ns netpol-demo
```
