
## 🛠️ **Übung: CronJob in Kubernetes**

### 🎯 Ziel

* Einen CronJob erstellen, der alle 30 Sekunden (zum Testen) eine Nachricht ausgibt.
* Logs des CronJobs prüfen.
* Verhalten bei Fehlschlägen und Parallelism beobachten.

---

## 📁 Schritt 1: CronJob erstellen

### Datei: `hello-cronjob.yaml`

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello-cronjob
spec:
  schedule: "*/1 * * * *"  # jede Minute
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - echo "📅 Hallo von CronJob um $(date)"
          restartPolicy: OnFailure
```

```bash
kubectl apply -f hello-cronjob.yaml
```

---

## 🔍 Schritt 2: Ausführungen beobachten

Warte 1–2 Minuten, dann:

```bash
kubectl get cronjob hello-cronjob
kubectl get jobs
kubectl get pods --selector=job-name --sort-by=.metadata.creationTimestamp
```

---

## 📂 Schritt 3: Logs eines CronJob-Runs ansehen

```bash
kubectl get pods  # z. B. hello-cronjob-xxxx
kubectl logs <PODNAME>
```

→ Beispielausgabe:

```
📅 Hallo von CronJob um Mon Jun  6 10:00:00 UTC 2025
```

---

## 🧪 Schritt 4: Fehler simulieren

Ändere den CronJob so, dass der Container einen Fehler produziert:

```yaml
args:
- /bin/sh
- -c
- exit 1
```

→ Füge auch ein `backoffLimit: 1` hinzu:

```yaml
spec:
  backoffLimit: 1
```

→ Neu anwenden:

```bash
kubectl apply -f hello-cronjob.yaml
```

Dann prüfen:

```bash
kubectl get jobs
kubectl describe job <JOBNAME>
```

→ Status zeigt **"failed"** nach 1 Versuch.

---

## 🧩 Optional: Ausführung manuell triggern

```bash
kubectl create job --from=cronjob/hello-cronjob manual-run
```

---

## 🧹 Aufräumen

```bash
kubectl delete cronjob hello-cronjob
kubectl delete jobs --all
```

---

## 🧠 Wichtiges Wissen

| Feld                         | Bedeutung                                          |
| ---------------------------- | -------------------------------------------------- |
| `schedule`                   | Cron-Syntax wie `* * * * *`                        |
| `restartPolicy`              | Muss bei CronJobs `OnFailure` oder `Never` sein    |
| `jobTemplate`                | Enthält den eigentlichen Task                      |
| `backoffLimit`               | Anzahl Wiederholungen bei Fehler                   |
| `successfulJobsHistoryLimit` | Anzahl erfolgreicher Job-Runs, die behalten werden |
| `failedJobsHistoryLimit`     | Dasselbe für Fehlversuche                          |

