
# 🧪 Übung: ReplicaSet verliert Kontrolle über Pod durch Label-Änderung

---

## 🎯 Ziel

* Beobachten, wie ein ReplicaSet auf eine **Labeländerung** eines verwalteten Pods reagiert
* Verstehen, wie wichtig `matchLabels` für die Replikation ist

---

## 📁 Schritt 1: ReplicaSet erstellen

```yaml
# rs-demo.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rs-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: demo
  template:
    metadata:
      labels:
        app: demo
    spec:
      containers:
        - name: nginx
          image: nginx
```

Anwenden:

```bash
kubectl apply -f rs-demo.yaml
```

---

## 📁 Schritt 2: Pod und Labels prüfen

```bash
kubectl get pods --show-labels
```

> Du siehst einen Pod mit dem Label `app=demo`.

---

## 📁 Schritt 3: Label ändern (Pod "entkoppeln")

```bash
POD=$(kubectl get pods -l app=demo -o jsonpath='{.items[0].metadata.name}')
kubectl label pod $POD app=foo --overwrite
```

---

## 📁 Schritt 4: Verhalten beobachten

```bash
kubectl get pods --show-labels
```

> 🔍 Du siehst jetzt **zwei Pods**:
>
> * Der alte Pod mit `app=foo` – wird **nicht mehr vom ReplicaSet verwaltet**
> * Ein **neuer Pod mit `app=demo`**, weil das ReplicaSet wieder einen Pod braucht, der seinem `selector` entspricht.

---

## 📁 Schritt 5: Nachvollziehen

```bash
kubectl describe rs rs-demo
```

* Unter `Pods Status` siehst du: 1 aktuell kontrollierter Pod
* Der „entkoppelte“ Pod zählt nicht mehr dazu
