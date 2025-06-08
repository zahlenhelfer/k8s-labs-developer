
## 🛠️ **Übung: Kubernetes Services – alle Typen im Vergleich**

### 🎯 Ziel

* Einen einfachen Webserver-Pod bereitstellen
* Alle vier Service-Typen erstellen und testen:

  * ClusterIP (Standard)
  * NodePort
  * LoadBalancer
  * ExternalName

---

## 🔧 Gemeinsame Grundlage: Der Webserver

### 📁 Datei: `webserver-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-pod
  labels:
    app: hello
spec:
  containers:
  - name: hello
    image: hashicorp/http-echo
    args:
      - "-text=Hallo vom Pod"
    ports:
      - containerPort: 5678
```

```bash
kubectl apply -f webserver-pod.yaml
```

---

## 🧩 Teil 1: ClusterIP (Standard)

### 📁 Datei: `service-clusterip.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-clusterip
spec:
  selector:
    app: hello
  ports:
  - protocol: TCP
    port: 80
    targetPort: 5678
  type: ClusterIP
```

```bash
kubectl apply -f service-clusterip.yaml
```

### ✅ Test (nur intern):

```bash
kubectl run curl --image=busybox -it --rm --restart=Never -- /bin/sh
# Im Container:
wget -qO- http://hello-clusterip
exit
```

---

## 🧩 Teil 2: NodePort

### 📁 Datei: `service-nodeport.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-nodeport
spec:
  type: NodePort
  selector:
    app: hello
  ports:
  - port: 80
    targetPort: 5678
    nodePort: 30080
```

```bash
kubectl apply -f service-nodeport.yaml
```

### ✅ Test (außerhalb des Clusters):

```bash
curl http://<Node-IP>:30080
```

> In Minikube:

```bash
minikube service hello-nodeport --url
```

---

## 🧩 Teil 3: LoadBalancer

### 📁 Datei: `service-loadbalancer.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-loadbalancer
spec:
  selector:
    app: hello
  ports:
  - port: 80
    targetPort: 5678
  type: LoadBalancer
```

```bash
kubectl apply -f service-loadbalancer.yaml
```

### ✅ Test:

```bash
kubectl get service hello-loadbalancer
```

→ Externe IP abwarten und dann:

```bash
curl http://<EXTERNAL-IP>
```

> In Minikube:

```bash
minikube tunnel
```

---

## 🧩 Teil 4: ExternalName

### 📁 Datei: `service-externalname.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-externalname
spec:
  type: ExternalName
  externalName: example.org
```

```bash
kubectl apply -f service-externalname.yaml
```

### ✅ Test:

```bash
kubectl run curl2 --image=busybox -it --rm --restart=Never -- /bin/sh
# Im Container:
wget -qO- http://hello-externalname
exit
```

---

## 🧹 Aufräumen

```bash
kubectl delete -f webserver-pod.yaml
kubectl delete svc hello-clusterip hello-nodeport hello-loadbalancer hello-externalname
```

---

## 🧠 Zusammenfassung der Service-Typen

| Typ              | Intern erreichbar | Extern erreichbar     | Anwendungsfall                    |
| ---------------- | ----------------- | --------------------- | --------------------------------- |
| **ClusterIP**    | ✅                 | ❌                     | Standard-Kommunikation im Cluster |
| **NodePort**     | ✅                 | ✅ (über Node-IP)      | Extern erreichbar ohne LB         |
| **LoadBalancer** | ✅                 | ✅ (öffentliche IP)    | Public-Service mit Cloud-LB       |
| **ExternalName** | ❌ (nur DNS)       | ✅ (per DNS-Auflösung) | DNS-Umleitung auf externen Host   |

