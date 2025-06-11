## 🛠️ **Übung: Kubernetes Deployment mit Rollout & Rollback**

### 🎯 Ziel

* Ein NGINX Deployment erstellen.
* Ein Update durchführen und Rollout-History ansehen.
* Einen Fehler provozieren und per **Rollback** zurückgehen.

---

### 📁 Schritt 1: `nginx-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  revisionHistoryLimit: 5
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
        image: nginx:1.21
        ports:
        - containerPort: 80
```

---

### 🚀 Schritt 2: Deployment starten

```bash
kubectl apply -f nginx-deployment.yaml
```

Dann prüfen:

```bash
kubectl get deployment nginx-deployment
kubectl get pods -l app=nginx
```

---

### 🔍 Schritt 3: Rollout-History prüfen

```bash
kubectl rollout history deployment nginx-deployment
```

→ Du siehst `REVISION 1`

---

### 🧪 Schritt 4: Update durchführen (z. B. neue NGINX-Version)

```bash
kubectl set image deployment nginx-deployment nginx=nginx:1.25
```

Dann prüfen:

```bash
kubectl rollout status deployment nginx-deployment
kubectl rollout history deployment nginx-deployment
```

→ Jetzt hast du `REVISION 2`

---

### 💥 Schritt 5: Fehlerhafte Version einspielen

```bash
kubectl set image deployment nginx-deployment nginx=nginx:doesnotexist
```

Prüfe:

```bash
kubectl rollout status deployment nginx-deployment
```

→ Rollout **hängt**, weil das Image nicht existiert. Bitte die Ausgabe mit `Control+C` verlassen

---

### ↩️ Schritt 6: Rollback

```bash
kubectl rollout undo deployment nginx-deployment
```

Danach prüfen:

```bash
kubectl get pods -l app=nginx
kubectl rollout history deployment nginx-deployment
```

→ Zurück auf `nginx:1.25` (REVISION 2) -> die dann als (REVISION 4) geführt wird

Optional: Rollback auf eine konkrete Revision:

```bash
kubectl rollout undo deployment nginx-deployment --to-revision=1
```

---

### 🧹 Schritt 7: Aufräumen

```bash
kubectl delete -f nginx-deployment.yaml
```

---

### 🧠 Wichtiges Wissen

| Befehl                    | Bedeutung                                                  |
| ------------------------- | ---------------------------------------------------------- |
| `kubectl rollout history` | Zeigt dir vergangene Revisionen                            |
| `kubectl rollout undo`    | Macht den letzten oder einen bestimmten Rollout rückgängig |
| `kubectl set image`       | Update des Deployments                                     |
| `revisionHistoryLimit`    | Wie viele alte Revisionen Kubernetes speichert             |

