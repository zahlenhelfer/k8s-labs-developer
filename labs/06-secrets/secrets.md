
## 🛠️ **Übung: Kubernetes Secret erstellen und im Pod verwenden**

### 🎯 Ziel

* Ein Secret mit Benutzerdaten (Benutzername + Passwort) erstellen.
* Das Secret **als Umgebungsvariablen** und **als Datei** in einen Pod einbinden.
* Den Zugriff im Container prüfen.

---

## 📁 Schritt 1: Secret erstellen (zwei Varianten)

### ✅ Variante A: YAML-Datei

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: demo-secret
type: Opaque
data:
  username: YWRtaW4=         # base64 für "admin"
  password: c2VjcmV0MTIz     # base64 für "secret123"
```

```bash
kubectl apply -f demo-secret.yaml
```

### ✅ Variante B: per CLI (schnell)

```bash
kubectl create secret generic demo-secret \
  --from-literal=username=admin \
  --from-literal=password=secret123
```

---

## 📁 Schritt 2: Pod mit Secret als Umgebungsvariable

### Datei: `secret-env-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-demo
spec:
  containers:
  - name: demo
    image: busybox
    command: [ "sh", "-c", "echo USER=$USERNAME && echo PASS=$PASSWORD && sleep 3600" ]
    env:
    - name: USERNAME
      valueFrom:
        secretKeyRef:
          name: demo-secret
          key: username
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: demo-secret
          key: password
```

```bash
kubectl apply -f secret-env-pod.yaml
kubectl logs secret-env-demo
```

→ Ausgabe:

```
USER=admin
PASS=secret123
```

---

## 📁 Schritt 3: Secret als Datei mounten

### Datei: `secret-volume-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-vol-demo
spec:
  containers:
  - name: demo
    image: busybox
    command: [ "sh", "-c", "echo Datei-Inhalte:; cat /etc/secret-volume/username; cat /etc/secret-volume/password; sleep 3600" ]
    volumeMounts:
    - name: secret-vol
      mountPath: "/etc/secret-volume"
      readOnly: true
  volumes:
  - name: secret-vol
    secret:
      secretName: demo-secret
```

```bash
kubectl apply -f secret-volume-pod.yaml
kubectl logs secret-vol-demo
```

→ Ausgabe:

```
Datei-Inhalte:
admin
secret123
```

---

## 🔍 Geheimnis prüfen

```bash
kubectl get secret demo-secret -o yaml
```

→ Daten sind **base64-codiert**, nicht verschlüsselt.

Optional zum Entschlüsseln:

```bash
echo YWRtaW4= | base64 -d    # ergibt: admin
```

---

## 🧹 Aufräumen

```bash
kubectl delete pod secret-env-demo secret-vol-demo
kubectl delete secret demo-secret
```

---

## 🧠 Wichtiges Wissen

| Thema               | Erklärung                                          |
| ------------------- | -------------------------------------------------- |
| `type: Opaque`      | Beliebige Schlüssel-Werte als Secret speichern     |
| Base64-Codierung    | Secrets sind **nicht verschlüsselt**, nur codiert! |
| Zugriff über `env`  | Ideal für Umgebungsvariablen wie Passwörter        |
| Zugriff über Volume | Gut für Files wie `.dockerconfig`, TLS-Certs, etc. |

