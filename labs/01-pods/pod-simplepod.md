# 🛠️ **Übung: Erstelle einen einfachen Pod `simple-nginx-pod` in Kubernetes**

## 🎯 Ziel

* Einen Pod mit dem Namen `simple-nginx-pod` erstellen.
* Das Standard-Nginx-Image verwenden.
* Den Status und die Logs des Pods prüfen.

---

## 🧩 Schritt 1: YAML-Datei für den Pod schreiben

Erstelle eine Datei namens `simple-nginx-pod.yaml` mit folgendem Inhalt:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-nginx-pod
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
    ports:
    - containerPort: 80
```

---

## 🧩 Schritt 2: Pod erstellen

```bash
kubectl apply -f simple-nginx-pod.yaml
```

---

## 🧩 Schritt 3: Pod-Status überprüfen

```bash
kubectl get pod simple-nginx-pod
```

Wenn der Pod läuft, sollte `STATUS` auf `Running` stehen.

---

## 🧩 Schritt 4: Details anzeigen

```bash
kubectl describe pod simple-nginx-pod
```

Hier siehst du Events, Container-Status und weitere Details.

---

## 🧩 Schritt 5: Logs anzeigen

```bash
kubectl logs simple-nginx-pod
```

Dies zeigt dir die Standard-Logs des Nginx-Webservers.

---

## 🧩 Schritt 6: Interaktiv in den Container gehen (optional)

```bash
kubectl exec -it simple-nginx-pod -- /bin/bash
```

Du kannst z. B. mit `curl localhost` im Container prüfen, ob Nginx antwortet.

---

## 🧩 Schritt 7: Testen, ob Nginx von außen erreichbar ist (optional)

Wenn du einen Port weiterleiten möchtest:

```bash
kubectl port-forward pod/simple-nginx-pod 8080:80
```

Dann kannst du im Browser `http://localhost:8080` aufrufen.

---

## 🧹 Schritt 8: Pod löschen

```bash
kubectl delete pod simple-nginx-pod
```

---

## 📚 Was hast Du gelernt

* Wie man einen Pod mit einem Container erstellt.
* Wie man Kubernetes-Ressourcen beschreibt, überprüft und mit ihnen interagiert.
* Wie man einfache Fehlerdiagnose und Portweiterleitung macht.
