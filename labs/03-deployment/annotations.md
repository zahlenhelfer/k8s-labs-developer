# 🛠️ Übung: Verwendung von `kubernetes.io/change-cause` in Deployments

---

## 🎯 Ziel

* Deployment mit `--record` bzw. `--annotation` ausführen
* Rollout History inklusive Change Cause anzeigen
* Rollback auf eine frühere Revision durchführen

---

## 📁 Schritt 1: Einfaches Deployment anlegen

```bash
kubectl create deployment my-app --image=nginx --replicas=2 \
  --dry-run=client -o yaml > my-app.yaml
```

Bearbeite die Datei `my-app.yaml` und füge **diese Annotation** hinzu:

```yaml
metadata:
  annotations:
    kubernetes.io/change-cause: "Initial deployment with nginx"
```

Deployment erstellen:

```bash
kubectl apply -f my-app.yaml
```

---

## 📁 Schritt 2: Erste Änderung – neues Image

Bearbeite das Deployment (z. B. `nginx:1.25.0`):

```bash
kubectl set image deployment/my-app nginx=nginx:1.25.0 \
  --record
```

> `--record` speichert automatisch den Befehl als `change-cause`.

---

## 📁 Schritt 3: Weitere Änderung mit eigenem Change Cause

```bash
kubectl annotate deployment my-app \
  kubernetes.io/change-cause="Update auf nginx 1.25.2 wegen Sicherheitsupdate" \
  --overwrite

kubectl set image deployment/my-app nginx=nginx:1.25.2
```

---

## 📁 Schritt 4: Rollout-History prüfen

```bash
kubectl rollout history deployment my-app
```

> Du solltest mehrere Revisionen mit `CHANGE-CAUSE` sehen.

Details anzeigen:

```bash
kubectl rollout history deployment my-app --revision=2
```

---

## 📁 Schritt 5: Rollback durchführen

```bash
kubectl rollout undo deployment my-app --to-revision=1
```

Oder zum vorherigen Zustand:

```bash
kubectl rollout undo deployment my-app
```

---

## TODO: Optional Change-Cause mittels ENV setzten
