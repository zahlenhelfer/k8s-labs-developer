
## 🛠️ **Übung: MySQL Backup per CronJob mit `mysqldump`**

### 🎯 Ziel

* Einen CronJob anlegen, der regelmäßig ein MySQL-Dump in ein PVC schreibt
* MySQL-Zugangsdaten sicher per Secret bereitstellen
* Backup-Dateien im Volume speichern und prüfen

---

## Vorbereitung: MySQL Deployment + Service

Falls du noch keinen MySQL-Server im Cluster hast, hier ein minimaler MySQL-Deployment mit Secret:

### 1. Secret mit MySQL-Zugangsdaten

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
stringData:
  MYSQL_ROOT_PASSWORD: rootpassword123
```

```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
stringData:
  MYSQL_ROOT_PASSWORD: rootpassword123
EOF
```

---

### 2. MySQL Deployment + Service

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: MYSQL_ROOT_PASSWORD
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-data
        emptyDir: {}  # für Demo, besser PVC in Produktion
---
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  selector:
    app: mysql
  ports:
  - port: 3306
    targetPort: 3306
```

```bash
kubectl apply -f mysql-deployment.yaml
```

---

## 3. PersistentVolumeClaim (PVC) für Backups

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-backup-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

```bash
kubectl apply -f pvc.yaml
```

---

## 4. CronJob für Backup

### Datei: `mysql-backup-cronjob.yaml`

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: mysql-backup
spec:
  schedule: "*/5 * * * *"  # alle 5 Minuten zum Testen
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: mysql:8.0
            env:
            - name: MYSQL_PWD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_ROOT_PASSWORD
            command:
            - /bin/sh
            - -c
            - |
              mkdir -p /backup && \
              mysqldump -h mysql -u root --all-databases > /backup/alldb-$(date +%F-%H%M%S).sql
            volumeMounts:
            - name: backup-volume
              mountPath: /backup
          restartPolicy: OnFailure
          volumes:
          - name: backup-volume
            persistentVolumeClaim:
              claimName: mysql-backup-pvc
```

```bash
kubectl apply -f mysql-backup-cronjob.yaml
```

---

## 5. Backup prüfen

Nach einigen Minuten:

```bash
kubectl get pods --selector=job-name=mysql-backup --watch
```

Dann in einen Backup-Pod rein:

```bash
kubectl get pods --selector=job-name=mysql-backup
kubectl exec -it <PODNAME> -- /bin/sh
ls -l /backup
cat /backup/alldb-2025-06-07-123456.sql | head
exit
```

---

## 6. Aufräumen

```bash
kubectl delete cronjob mysql-backup
kubectl delete pvc mysql-backup-pvc
kubectl delete deployment mysql
kubectl delete service mysql
kubectl delete secret mysql-secret
```

---

## 🧠 Wissenswertes

| Punkt                           | Erläuterung                                  |
| ------------------------------- | -------------------------------------------- |
| Secret mit Root-Passwort        | Sicherer Umgang mit Zugangsdaten             |
| PVC für Backups                 | Persistente Speicherung von Backupdateien    |
| CronJob Timing                  | Flexibel, z.B. täglich, stündlich, minütlich |
| `mysqldump` CLI-Flags           | Für gezielte Backups (z.B. einzelne DBs)     |
| Backup-Speicherort im Container | VolumeMount auf `/backup`                    |
