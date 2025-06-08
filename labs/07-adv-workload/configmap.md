
## 🛠️ **Übung: ConfigMap erstellen und im Pod verwenden**

### 🎯 Ziel

* Eine ConfigMap mit Konfigurationswerten erstellen.
* Diese ConfigMap als Umgebungsvariable und als Datei in einen Pod einbinden.
* Den Inhalt im Container anzeigen.

---

### 📁 Schritt 1: Erstelle die Datei `my-configmap.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: demo-config
data:
  APP_MODE: "development"
  WELCOME_MSG: "Willkommen in Kubernetes!"
```

---

### 🚀 Schritt 2: ConfigMap anwenden

```bash
kubectl apply -f my-configmap.yaml
```

---

### 📁 Schritt 3: Erstelle Datei `pod-with-configmap.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-demo
spec:
  containers:
  - name: demo-container
    image: busybox
    command: [ "sh", "-c", "env; echo; echo Datei-Inhalt:; cat /etc/config/WELCOME_MSG; sleep 3600" ]
    env:
    - name: APP_MODE
      valueFrom:
        configMapKeyRef:
          name: demo-config
          key: APP_MODE
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: demo-config
```

---

### 🚀 Schritt 4: Pod starten

```bash
kubectl apply -f pod-with-configmap.yaml
```

---

### 🔍 Schritt 5: Pod prüfen

```bash
kubectl get pod configmap-demo
kubectl logs configmap-demo
```

→ Du solltest Folgendes sehen:

* `APP_MODE=development` aus Umgebungsvariable
* Datei-Inhalt aus `/etc/config/WELCOME_MSG`: `Willkommen in Kubernetes!`

---

### 🧪 Schritt 6: Werte live ändern

Ändere z. B. den Begrüßungstext:

```bash
kubectl create configmap demo-config --from-literal=WELCOME_MSG="Hallo aus dem Cluster!" --from-literal=APP_MODE="production" -o yaml --dry-run=client | kubectl apply -f -
```

Danach:

```bash
kubectl delete pod configmap-demo
kubectl apply -f pod-with-configmap.yaml
```

→ Änderungen sind **nur nach Neustart** des Pods wirksam!

---

### 🧹 Schritt 7: Aufräumen

```bash
kubectl delete pod configmap-demo
kubectl delete configmap demo-config
```

---

### 🧠 Wichtiges Wissen

| Verwendung        | Beschreibung                                                     |
| ----------------- | ---------------------------------------------------------------- |
| **envFrom / env** | Zugriff auf Werte als Umgebungsvariablen                         |
| **volumeMount**   | Zugriff auf Werte als Dateien im Container                       |
| **Live-Updates**  | Werden nicht automatisch erkannt – Pod muss neu gestartet werden |

---

