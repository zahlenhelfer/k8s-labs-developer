
# 🛠️ Übung: Einfaches Helm-Chart erstellen und deployen

---

## 🎯 Ziel

* Helm installieren und konfigurieren
* Ein eigenes Helm-Chart erstellen
* Deployment einer NGINX-App mit Helm
* Werte anpassen und Chart upgraden

---

## 📁 Schritt 1: Helm installieren

Falls noch nicht installiert, [Helm installieren](https://helm.sh/docs/intro/install/).

Prüfe:

```bash
helm version
```

---

## 📁 Schritt 2: Neues Helm-Chart erstellen

```bash
helm create my-nginx-chart
```

Dies legt einen Chart-Ordner `my-nginx-chart` mit Standardstruktur an.

---

## 📁 Schritt 3: Chart anpassen

* Öffne `my-nginx-chart/values.yaml`
* Passe z. B. den `replicaCount` an (z. B. auf 2)

```yaml
replicaCount: 2
```

* Optional: Unter `image` kannst du das NGINX-Image oder Tag ändern.

---

## 📁 Schritt 4: Deployment testen

Im Kubernetes-Cluster (minikube, kind o.ä.):

```bash
kubectl create namespace demo
helm install my-nginx my-nginx-chart --namespace demo
```

---

## 📁 Schritt 5: Deployment prüfen

```bash
kubectl get pods -n demo
kubectl get svc -n demo
```

---

## 📁 Schritt 6: Werte ändern und Chart upgraden

Zum Beispiel Anzahl Replikate auf 3 ändern:

```bash
helm upgrade my-nginx my-nginx-chart --namespace demo --set replicaCount=3
```

---

## 📁 Schritt 7: Release löschen

```bash
helm uninstall my-nginx --namespace demo
```

