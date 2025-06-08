
## 🛠️ Übung: Persistent Volume (PV) und Persistent Volume Claim (PVC) in Kubernetes

### 🎯 Ziel

* Ein statisches Persistent Volume (PV) anlegen
* Einen Persistent Volume Claim (PVC) erstellen, der das PV beansprucht
* Einen Pod starten, der das PVC als Volume verwendet und Daten darin speichert

---

## 📁 Schritt 1: Persistent Volume anlegen

Erstelle eine Datei `pv.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data
  persistentVolumeReclaimPolicy: Retain
```

> **Hinweis:**
> `hostPath` funktioniert nur in lokalen Clustern (z.B. Minikube, kind).
> In Cloud-Umgebungen nutzt du meist andere Storage Klassen.

---

## 📁 Schritt 2: Persistent Volume Claim anlegen

Erstelle eine Datei `pvc.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

---

## 📁 Schritt 3: Pod mit PVC verwenden

Erstelle eine Datei `pod-with-pvc.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-pvc
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sleep", "3600"]
    volumeMounts:
    - mountPath: /data
      name: storage
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: my-pvc
```

---

## 📁 Schritt 4: Ressourcen anwenden und prüfen

```bash
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml
kubectl apply -f pod-with-pvc.yaml
```

Prüfe Status:

```bash
kubectl get pv
kubectl get pvc
kubectl get pod pod-with-pvc
```

---

## 📁 Schritt 5: Daten im Volume speichern (optional)

```bash
kubectl exec -it pod-with-pvc -- sh
```

Im Container:

```sh
cd /data
echo "Hallo PV!" > testfile.txt
cat testfile.txt
exit
```

---

## 📁 Schritt 6: Aufräumen

```bash
kubectl delete pod pod-with-pvc
kubectl delete pvc my-pvc
kubectl delete pv my-pv
```

---

## 🧠 Erklärung

| Begriff                       | Bedeutung                                                     |
| ----------------------------- | ------------------------------------------------------------- |
| PersistentVolume (PV)         | Speicher-Ressource im Cluster, z.B. ein Datenträger           |
| PersistentVolumeClaim (PVC)   | Anfrage eines Pods nach Speicher mit bestimmten Eigenschaften |
| accessModes                   | Wie das Volume verwendet werden darf (z.B. ReadWriteOnce)     |
| hostPath                      | Lokaler Pfad auf dem Node (nur Test/Dev!)                     |
| persistentVolumeReclaimPolicy | Verhalten bei Löschung des PVC (Retain, Delete, Recycle)      |

