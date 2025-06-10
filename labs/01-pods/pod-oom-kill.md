# 💥 **Erweiterte Übung: OOMKill (Out Of Memory Kill) provozieren**

## 🎯 Ziel

* Einen Pod mit einer **bewusst niedrigen Speichergrenze** erstellen.
* Ein Skript ausführen, das zu viel RAM nutzt.
* Beobachten, wie Kubernetes den Container killt.

---

## 📁 Schritt 1: YAML-Datei `oomkill-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: oom-demo
spec:
  containers:
  - name: memory-hog
    image: python:3.12-slim
    command: ["python3", "-c", "a = ['x' * 1024 * 1024 * 1024] * 200; import time; time.sleep(30)"]
    resources:
      limits:
        memory: "50Mi"
        cpu: "100m"
```

**Erklärung:**

* Das Python-Kommando erstellt eine Liste mit \~200 MB an Daten.
* Der Speicher-Limit liegt aber bei nur **50 MiB**.
* Das führt zu einem **OOMKill durch das Kubernetes-System**.

---

## 🚀 Schritt 2: Pod starten

```bash
kubectl apply -f oomkill-pod.yaml
```

---

## ⏱️ Schritt 3: Beobachte das Verhalten

Gib dem Pod ein paar Sekunden, dann prüfe den Status:

```bash
kubectl get pod oom-demo
kubectl describe pod oom-demo
```

Du solltest sehen:

```
State:          Terminated
Reason:         OOMKilled
```

---

## 🧠 Schritt 4: Verständnis vertiefen

```bash
kubectl get pod oom-demo -o jsonpath='{.status.containerStatuses[0].lastState.terminated.reason}'
```

Gibt `"OOMKilled"` zurück, wenn Kubernetes den Container aufgrund von zu hohem Speicherverbrauch beendet hat.

---

## 🧹 Schritt 5: Aufräumen

```bash
kubectl delete pod oom-demo
```

---

## 📚 Was Du hier gelernt hast

| Konzept            | Bedeutung                                                                                    |
| ------------------ | -------------------------------------------------------------------------------------------- |
| **OOMKilled**      | Container wurde durch das Betriebssystem beendet, da er das Speicherlimit überschritten hat. |
| **Limits**         | Ressourcen-Grenzen sind real – beim Überschreiten greift Kubernetes hart durch.              |
| **Fehlerdiagnose** | Du lernst zu erkennen, ob ein Container durch OOM abgeschossen wurde.                        |
