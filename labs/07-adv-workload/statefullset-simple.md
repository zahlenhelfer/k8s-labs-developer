## 🛠️ **Übung: StatefulSet mit NGINX und persistentem Speicher**

### 🎯 Ziel

* Einen **StatefulSet** mit mehreren NGINX-Replikas erstellen.
* Jeder Pod bekommt einen **persistenten VolumeClaim**, der eindeutig ist.
* Die stabilen Netzwerknamen und Volumes der StatefulSet-Pods beobachten.

---

### 📚 Was du brauchst

* Eine laufende Kubernetes-Umgebung.
* Ein StorageClass-fähiger Cluster (z. B. Minikube oder Kind mit Standard-StorageClass).

---

### 📁 Schritt 1: `nginx-statefulset.yaml`

Erstelle die folgende Datei:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  clusterIP: None
  selector:
    app: nginx
  ports:
  - name: http
    port: 80
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  serviceName: "nginx"
  replicas: 3
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
          name: http
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

---

### 🧩 Schritt 2: StatefulSet anwenden

```bash
kubectl apply -f nginx-statefulset.yaml
```

---

### 🔍 Schritt 3: Ergebnisse prüfen

#### Pods:

```bash
kubectl get pods -l app=nginx
```

→ Du solltest sehen:

```
nginx-0
nginx-1
nginx-2
```

#### PVCs:

```bash
kubectl get pvc
```

→ Du solltest sehen:

```
www-nginx-0
www-nginx-1
www-nginx-2
```

Jeder Pod hat **einen eigenen persistenten Speicher**.

---

### 🧪 Schritt 4: Schreibtest auf dem Volume

```bash
kubectl exec nginx-0 -- sh -c "echo nginx-0 > /usr/share/nginx/html/index.html"
```

Dann:

```bash
kubectl port-forward pod/nginx-0 8080:80
```

Besuche im Browser: [http://localhost:8080](http://localhost:8080) → du solltest „nginx-0“ sehen.

---

### 🔁 Schritt 5: Pod löschen und prüfen, ob Volume erhalten bleibt

```bash
kubectl delete pod nginx-0
```

Warte, bis er neu gestartet wurde, dann erneut aufrufen:

```bash
kubectl port-forward pod/nginx-0 8080:80
```

Du solltest **immer noch „nginx-0“** sehen → das zeigt, dass **persistenter Speicher erhalten** bleibt.

---

### 🧹 Schritt 6: Aufräumen

```bash
kubectl delete -f nginx-statefulset.yaml
kubectl delete pvc -l app=nginx
```

---

### 🧠 Bonus-Wissen

| Element                  | Besonderheit bei StatefulSet                                 |
| ------------------------ | ------------------------------------------------------------ |
| **Name**                 | Pods heißen `nginx-0`, `nginx-1`, … statt zufälliger Namen   |
| **VolumeClaimTemplates** | Jeder Pod bekommt ein eigenes PVC                            |
| **Stable Network ID**    | `nginx-0.nginx` ist ein DNS-Name, der nur für `nginx-0` gilt |

