
# 🛠️ Übung: Helm-Chart, das ein Kubernetes Secret erstellt

---

## 🎯 Ziel

* Ein Helm-Chart erstellen
* Ein Kubernetes Secret via Template anlegen
* Secret im Deployment als Umgebungsvariable nutzen

---

## 📁 Schritt 1: Neues Helm-Chart anlegen

```bash
helm create secret-demo
cd secret-demo
```

---

## 📁 Schritt 2: Secret Template anlegen

Erstelle Datei `templates/secret.yaml` mit folgendem Inhalt:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: {{ include "secret-demo.fullname" . }}-secret
type: Opaque
data:
  # base64-kodierte Werte
  username: {{ .Values.secret.username | b64enc | quote }}
  password: {{ .Values.secret.password | b64enc | quote }}
```

---

## 📁 Schritt 3: Werte in values.yaml definieren

Füge in `values.yaml` folgendes hinzu:

```yaml
secret:
  username: admin
  password: s3cr3tpass
```

---

## 📁 Schritt 4: Deployment anpassen, um Secret zu nutzen

Öffne `templates/deployment.yaml` und ergänze unter `spec.template.spec.containers[0]` folgendes (innerhalb `env:`):

```yaml
env:
  - name: USERNAME
    valueFrom:
      secretKeyRef:
        name: {{ include "secret-demo.fullname" . }}-secret
        key: username
  - name: PASSWORD
    valueFrom:
      secretKeyRef:
        name: {{ include "secret-demo.fullname" . }}-secret
        key: password
```

Falls `env:` noch nicht da ist, lege es an.

---

## 📁 Schritt 5: Chart deployen

```bash
helm install my-secret-demo . 
```

---

## 📁 Schritt 6: Prüfen

* Secret anschauen:

```bash
kubectl get secret my-secret-demo-secret -o yaml
```

* Pod-Umgebungsvariablen prüfen (z.B. mit `kubectl exec` oder Logs).

---

## 📁 Schritt 7: Werte anpassen & Upgrade

```bash
helm upgrade my-secret-demo . --set secret.password=newpass123
```

---

## Zusammenfassung

Mit Helm kannst du Kubernetes-Secrets bequem als Template erstellen und dynamisch Werte aus `values.yaml` einfügen. Damit lassen sich sensible Daten sicher verwalten und in Deployments verwenden.
