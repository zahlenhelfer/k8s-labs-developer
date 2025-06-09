
# 🧪 Übung: Arbeiten mit Helm Charts in Kubernetes

---

## 🎯 Ziel

* Helm installieren (falls nötig)
* Ein öffentliches Helm-Chart (z. B. `bitnami/nginx`) verwenden
* Installation mit benutzerdefinierten Werten
* Werte inspizieren und anpassen
* Helm-Release wieder entfernen

---

## 🧰 Voraussetzungen

* Ein laufender Kubernetes-Cluster (z. B. via Minikube, Kind, etc.)
* Helm installiert:

  ```bash
  helm version
  ```

Wenn nicht, installiere es:
[https://helm.sh/docs/intro/install/](https://helm.sh/docs/intro/install/)

---

## 📦 1. Helm-Repository hinzufügen

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

---

## 🔍 2. Chart-Infos ansehen

```bash
helm search repo bitnami/nginx
```

Optional: zeige die Standardwerte (Values):

```bash
helm show values bitnami/nginx > nginx-values.yaml
```

---

## 🚀 3. Helm-Chart installieren

```bash
helm install my-nginx bitnami/nginx --set service.type=NodePort
```

Wichtig:

* `my-nginx` ist der Release-Name
* `bitnami/nginx` ist das Chart
* `--set` erlaubt dir Werte zu überschreiben

---

## 🔍 4. Ressourcen anzeigen

```bash
kubectl get all
```

> Du siehst: Pod, Service, Deployment, ReplicaSet

Optional: die Werte des Releases inspizieren

```bash
helm get values my-nginx
```

---

## ✏️ 5. Helm-Chart aktualisieren

Du kannst Werte ändern z. B. über ein eigenes File:

**nginx-custom-values.yaml**

```yaml
service:
  type: NodePort
  nodePorts:
    http: 30080
```

Dann anwenden:

```bash
helm upgrade my-nginx bitnami/nginx -f nginx-custom-values.yaml
```

---

## 🧪 6. Zugriff testen

```bash
minikube service my-nginx --url
```

Oder (wenn du kein Minikube nutzt):

```bash
kubectl get svc my-nginx
# Dann manuell zu http://<NodeIP>:<NodePort> surfen
```

---

## 🗑️ 7. Helm-Release deinstallieren

```bash
helm uninstall my-nginx
```

Optional: Cache & Repo aufräumen

```bash
helm repo remove bitnami
```

---

## 🧠 Fazit

In dieser Übung hast du gelernt:

* Wie man ein Helm-Chart sucht, installiert und anpasst
* Wie man ein Release updated und wieder löscht
* Wie man mit `--set` oder `-f` Konfigurationen überschreibt
* Helm = mächtiger Paketmanager für Kubernetes
