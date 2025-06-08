
# 🛠️ Übung: Kubernetes Pod mit `hostPath` Volume

---

## 🎯 Ziel

* Einen Pod erstellen, der ein Volume vom Typ `hostPath` einbindet
* Dateien auf dem Host-System vom Container aus lesen/schreiben
* Das Verhalten von `hostPath` verstehen

---

## 📁 Schritt 1: `hostPath`-Pod YAML erstellen

Erstelle die Datei `hostpath-pod.yaml` mit folgendem Inhalt:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-demo
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep", "3600"]
    volumeMounts:
    - mountPath: /data
      name: my-volume
  volumes:
  - name: my-volume
    hostPath:
      path: /tmp/hostdata
      type: DirectoryOrCreate
```

**Erläuterung:**

* `hostPath.path: /tmp/hostdata`: Dieser Pfad wird auf dem **Node (nicht im Container!)** verwendet.
* `DirectoryOrCreate`: Wenn `/tmp/hostdata` auf dem Node nicht existiert, wird es automatisch erstellt.
* Im Container ist der Pfad unter `/data` verfügbar.

---

## 📁 Schritt 2: Pod anwenden

```bash
kubectl apply -f hostpath-pod.yaml
```

---

## 📁 Schritt 3: In den Container einsteigen

```bash
kubectl exec -it hostpath-demo -- bash
```

Dann im Container:

```bash
echo "Hello from inside the container" > /data/test.txt
exit
```

---

## 📁 Schritt 4: Auf dem Host (Node) prüfen

> Voraussetzung: Du kannst auf den Node zugreifen (z. B. bei minikube oder kind möglich)

```bash
minikube ssh
cat /tmp/hostdata/test.txt
```

Du solltest sehen:

```
Hello from inside the container
```

---

## 📁 Schritt 5: Clean-up

```bash
kubectl delete pod hostpath-demo
```

Optional: Auf dem Host den Ordner löschen

```bash
minikube ssh
sudo rm -rf /tmp/hostdata
```

