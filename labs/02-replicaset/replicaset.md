---

## 🛠️ **Übung: ReplicaSet mit NGINX**

### 🎯 Ziel

* Ein ReplicaSet erstellen, das **3 identische NGINX-Pods** betreibt.
* Den automatischen Ersatz von Pods beobachten.

---

### 📁 Schritt 1: `nginx-replicaset.yaml`

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
spec:
  replicas: 3
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
        image: nginx:latest
        ports:
        - containerPort: 80
```

---

### 🚀 Schritt 2: ReplicaSet anwenden

```bash
kubectl apply -f nginx-replicaset.yaml
```

---

### 🔍 Schritt 3: Ergebnisse prüfen

```bash
kubectl get replicaset
kubectl get pods -l app=nginx
```

→ Du siehst: 3 Pods mit zufälligen Namen, z. B. `nginx-replicaset-abc123`.

---

### 🧪 Schritt 4: Verhalten testen

Lösche einen der Pods:

```bash
kubectl delete pod <pod-name>
```

Dann erneut prüfen:

```bash
kubectl get pods
```

→ Kubernetes erstellt **sofort einen neuen**, um die Anzahl bei 3 zu halten.

---

### 🛠️ Schritt 5: Anzahl der Replikas ändern

```bash
kubectl scale replicaset nginx-replicaset --replicas=5
```

→ Es werden **2 zusätzliche Pods** gestartet.

---

### 🧹 Schritt 6: Alles löschen

```bash
kubectl delete -f nginx-replicaset.yaml
```

---

### 🧠 Wichtiges Verständnis

| Konzept                        | Erklärung                                                                            |
| ------------------------------ | ------------------------------------------------------------------------------------ |
| **ReplicaSet**                 | Sorgt dafür, dass zu jeder Zeit die definierte Anzahl von Pods aktiv ist.            |
| **Pod-Template**               | Das Muster für neue Pods – ReplicaSet erstellt daraus Kopien.                        |
| **Nicht für Rollouts gedacht** | ReplicaSets sind **nicht für Updates** gebaut – dafür verwendet man **Deployments**. |

