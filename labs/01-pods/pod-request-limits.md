## 🛠️ **Übung: Resource Requests & Limits für einen Pod setzen**

### 🎯 Ziel

* Einen Pod mit **CPU- und Speicher-Limits** erstellen.
* Die Auswirkungen der Ressourcenbeschränkung beobachten.
* Die Konfiguration mit `kubectl` prüfen.

---

### 📁 Schritt 1: Erstelle eine Datei `resource-limits-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: limited-nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    resources:
      requests:
        memory: "64Mi"
        cpu: "100m"
      limits:
        memory: "128Mi"
        cpu: "200m"
    ports:
    - containerPort: 80
```

**Erklärung:**

* **Requests**: Garantierte minimale Ressourcen.
* **Limits**: Maximal erlaubte Ressourcen.
* `100m` CPU = 0.1 CPU-Kerne.

---

### 🚀 Schritt 2: Pod anwenden

```bash
kubectl apply -f resource-limits-pod.yaml
```

---

### 🔍 Schritt 3: Status und Ressourcen prüfen

```bash
kubectl get pod limited-nginx-pod
kubectl describe pod limited-nginx-pod
```

Suche im Output nach dem Abschnitt `Limits` und `Requests`.

---

### 🧪 Schritt 4: Ressourcenverhalten testen (optional)

Installiere `stress` im Container (nur zu Testzwecken):

```bash
kubectl exec -it limited-nginx-pod -- apt update
kubectl exec -it limited-nginx-pod -- apt install -y stress
kubectl exec -it limited-nginx-pod -- stress --cpu 2 --timeout 30
```

→ Du wirst feststellen, dass die CPU durch das Limit auf 200m beschränkt wird. Im echten Cluster mit aktivem **Kubelet und Cgroups** wird der Container "gedrosselt".

---

### 📈 Bonus: Nutzung überwachen (mit Metrics-Server)

Wenn du den [metrics-server](https://github.com/kubernetes-sigs/metrics-server) installiert hast:

```bash
kubectl top pod limited-nginx-pod
```

→ Zeigt aktuelle CPU- und Speicherwerte des Pods.

---

### 🧹 Schritt 5: Aufräumen

```bash
kubectl delete pod limited-nginx-pod
```

---

### 📚 Was du lernst

* Unterschied zwischen `requests` (Garantien) und `limits` (Obergrenzen).
* Wie Kubernetes Ressourcen verwaltet.
* Warum Limits wichtig sind (Stabilität, Fairness, Node-Auslastung).
