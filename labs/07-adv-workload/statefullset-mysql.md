
## 🛠️ **Übung: MySQL mit StatefulSet und PersistentVolumeClaims**

### 🎯 Ziel

* Eine MySQL-Datenbank als StatefulSet betreiben.
* Sicherstellen, dass jeder Pod eigenen Speicher hat.
* Stabilen Netzwerknamen und Datenpersistenz prüfen.

---

### ⚙️ Voraussetzungen

* Kubernetes-Cluster mit funktionierender StorageClass (z. B. Minikube).
* Keine bestehende MySQL-Instanz mit gleichem Namen im Namespace.

---

### 📁 Schritt 1: Erstelle die Datei `mysql-statefulset.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
  - port: 3306
    name: mysql
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  serviceName: "mysql"
  replicas: 3
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: my-secret-pw
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

---

### 🚀 Schritt 2: StatefulSet deployen

```bash
kubectl apply -f mysql-statefulset.yaml
```

---

### 🔍 Schritt 3: Überprüfen

#### Pods:

```bash
kubectl get pods -l app=mysql
```

→ Du siehst: `mysql-0`, `mysql-1`, `mysql-2`

#### PVCs:

```bash
kubectl get pvc
```

→ Du siehst: `mysql-data-mysql-0`, `mysql-data-mysql-1`, etc.

---

### 🧪 Schritt 4: Verbindung testen

Greife auf einen Pod zu:

```bash
kubectl exec -it mysql-0 -- mysql -u root -p
```

Passwort: `my-secret-pw`

Erstelle eine Test-Datenbank:

```sql
CREATE DATABASE testdb;
SHOW DATABASES;
EXIT;
```

---

### 🔁 Schritt 5: Pod löschen und prüfen, ob Daten erhalten bleiben

```bash
kubectl delete pod mysql-0
```

Warte, bis `mysql-0` neu gestartet wurde, und gehe wieder rein:

```bash
kubectl exec -it mysql-0 -- mysql -u root -p
```

→ `testdb` ist **noch da** – Beweis für persistente Speicherung!

---

### 🧹 Schritt 6: Aufräumen

```bash
kubectl delete -f mysql-statefulset.yaml
kubectl delete pvc -l app=mysql
```

---

### 🧠 Bonuswissen

| Element             | Erklärung                                                                                            |
| ------------------- | ---------------------------------------------------------------------------------------------------- |
| **clusterIP: None** | Headless Service – Pods können sich direkt über DNS `mysql-0.mysql`, `mysql-1.mysql` etc. ansprechen |
| **StatefulSet**     | Behält Pod-Namen und zugeordnete Volumes über Neustarts hinweg                                       |
| **PVCs**            | Werden automatisch für jeden Pod erstellt (eine pro Instanz)                                         |

