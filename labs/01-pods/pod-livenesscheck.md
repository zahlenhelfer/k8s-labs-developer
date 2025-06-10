## 🛠️ **Übung: Erstelle einen Pod mit Liveness-Probe in Kubernetes**

## 🎯 Ziel

* Einen einfachen Pod mit einem HTTP-Service erstellen.
* Eine **Liveness-Probe** konfigurieren, die die Verfügbarkeit des Containers überwacht.
* Den Liveness-Check testen, indem du absichtlich den Check fehlschlagen lässt.

---

## 🧩 Schritt 1: YAML-Datei für den Pod schreiben

Erstelle eine Datei namens `liveness-pod.yaml` mit folgendem Inhalt:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-demo
spec:
  containers:
  - name: webserver
    image: nginx
    ports:
    - containerPort: 80
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 10
```

**Erläuterung:**

* `httpGet` prüft den Pfad `/` auf Port 80.
* `initialDelaySeconds`: Wartezeit vor dem ersten Check.
* `periodSeconds`: Intervall für wiederholte Checks.

---

## 🧩 Schritt 2: Pod erstellen

```bash
kubectl apply -f liveness-pod.yaml
```

---

## 🧩 Schritt 3: Status prüfen

```bash
kubectl describe pod liveness-demo
kubectl get pod liveness-demo
```

Beobachte, ob der Pod stabil läuft.

---

## 🧪 Schritt 4: Liveness absichtlich fehlschlagen lassen

Um den Check scheitern zu lassen, kannst du in den laufenden Pod eingreifen:

```bash
kubectl exec -it liveness-demo -- /bin/bash
```

Ersetze die nginx-Startseite durch etwas anderes oder lösche sie:

```bash
rm /usr/share/nginx/html/index.html
exit
```

Jetzt sollte der Liveness-Check fehlschlagen, und Kubernetes startet den Container neu.

---

## 🔍 Schritt 5: Neustarts beobachten

```bash
kubectl get pod liveness-demo
```

Du solltest sehen, dass sich die Spalte `RESTARTS` erhöht.

---

## 🧹 Aufräumen: Pod löschen

```bash
kubectl delete pod liveness-demo
```
