
# 🧪 Übung: Dynamisches Provisionieren mit externem NFS-Share

---

## 📘 Voraussetzungen

* Der NFS-Server ist von deinen Kubernetes-Nodes aus **netzwerkseitig erreichbar**.
* Du nutzt Kubernetes >= 1.20
* Du hast `nfs-subdir-external-provisioner` installiert oder willst ihn jetzt installieren (wird gleich erklärt).

---

## ✅ Ziel

* Ein StorageClass-Objekt für NFS definieren
* NFS-Provisioner installieren (als dynamischer Storage-Broker)
* Eine PVC erstellen, die automatisch ein Verzeichnis unter `/student-share` nutzt
* Einen Pod starten, der das Volume nutzt

---

## 📁 Schritt 1: NFS Provisioner installieren

Verwende den offiziellen Helm-Chart:

```bash
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm repo update
```

Dann installieren wir den Provisioner in einem eigenen Namespace:

```bash
kubectl create namespace nfs-storage

helm install nfs-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
  --namespace nfs-storage \
  --set nfs.server=nfs.dockerlabs.de \
  --set nfs.path=/student-share \
  --set storageClass.name=nfs-client \
  --set storageClass.defaultClass=false
```

Das erstellt:

* eine `Deployment` für den Provisioner
* eine `StorageClass` namens `nfs-client`
* dynamische Volume-Verzeichnisse unter `/student00` für jede PVC

---

## 📁 Schritt 2: PersistentVolumeClaim anlegen

**pvc-nfs.yaml**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: nfs-client
```

```bash
kubectl apply -f pvc-nfs.yaml
```

---

## 📁 Schritt 3: Pod mit gemountetem Volume

**nfs-test-pod.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nfs-test
spec:
  containers:
  - name: test-container
    image: busybox
    command: ["/bin/sh", "-c", "while true; do date >> /mnt/nfs/dates.log; sleep 10; done"]
    volumeMounts:
    - mountPath: /mnt/nfs
      name: nfs-vol
  volumes:
  - name: nfs-vol
    persistentVolumeClaim:
      claimName: nfs-pvc
```

```bash
kubectl apply -f nfs-test-pod.yaml
```

---

## 📁 Schritt 4: Überprüfen

```bash
kubectl exec -it nfs-test -- cat /mnt/nfs/dates.log
```

> Du solltest fortlaufend neue Timestamps sehen – das zeigt, dass der Pod **schreibend** auf das Volume zugreifen kann.

---

## 📁 Optional: Standard-StorageClass setzen

Falls du möchtest, dass `nfs-client` die *Default*-StorageClass wird:

```bash
kubectl patch storageclass nfs-client -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

---

## 🧹 Aufräumen

```bash
kubectl delete pod nfs-test
kubectl delete pvc nfs-pvc
helm uninstall nfs-provisioner -n nfs-storage
kubectl delete namespace nfs-storage
```

