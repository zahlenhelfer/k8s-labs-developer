
# 🧪 Übung: EFK-Stack in Kubernetes für zentrales Logging

---

## 🎯 Ziel

* Deploye **Elasticsearch**, **Fluentd** (als DaemonSet), und **Kibana**
* Erfasse Container-Logs von allen Nodes
* Stelle sie visuell über Kibana dar
* Alles läuft innerhalb deines Kubernetes-Clusters

---

## 🛠️ Voraussetzungen

* Ein laufender Kubernetes-Cluster (z. B. Minikube oder Kind)
* Cluster-Ressourcen von mindestens 2 GB RAM (Elasticsearch braucht etwas!)

---

## 📁 Schritt 1: Namespace vorbereiten

```bash
kubectl create namespace logging
```

---

## 📁 Schritt 2: Elasticsearch Deployment

**elasticsearch.yaml**

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: elasticsearch
  namespace: logging
spec:
  serviceName: "elasticsearch"
  replicas: 1
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
    spec:
      containers:
      - name: elasticsearch
        image: docker.elastic.co/elasticsearch/elasticsearch:7.17.13
        ports:
        - containerPort: 9200
        env:
        - name: discovery.type
          value: single-node
        resources:
          limits:
            memory: "1Gi"
          requests:
            memory: "512Mi"
---
apiVersion: v1
kind: Service
metadata:
  name: elasticsearch
  namespace: logging
spec:
  ports:
  - port: 9200
    targetPort: 9200
  selector:
    app: elasticsearch
```

```bash
kubectl apply -f elasticsearch.yaml
```

---

## 📁 Schritt 3: Kibana Deployment

**kibana.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kibana
  namespace: logging
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kibana
  template:
    metadata:
      labels:
        app: kibana
    spec:
      containers:
      - name: kibana
        image: docker.elastic.co/kibana/kibana:7.17.13
        ports:
        - containerPort: 5601
        env:
        - name: ELASTICSEARCH_HOSTS
          value: http://elasticsearch:9200
---
apiVersion: v1
kind: Service
metadata:
  name: kibana
  namespace: logging
spec:
  type: NodePort
  ports:
    - port: 5601
      targetPort: 5601
      nodePort: 30001
  selector:
    app: kibana
```

```bash
kubectl apply -f kibana.yaml
```

---

## 📁 Schritt 4: Fluentd als DaemonSet

**fluentd.yaml**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluentd-config
  namespace: logging
data:
  fluent.conf: |
    <source>
      @type tail
      path /var/log/containers/*.log
      pos_file /fluentd/log/fluentd-containers.log.pos
      tag kube.*
      format json
    </source>

    <match **>
      @type elasticsearch
      host elasticsearch
      port 9200
      logstash_format true
      logstash_prefix kubernetes
      include_tag_key true
      type_name access_log
      flush_interval 5s
    </match>
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: logging
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd:v1.14-1
        env:
          - name: FLUENTD_CONF
            value: fluent.conf
        volumeMounts:
          - name: varlog
            mountPath: /var/log
          - name: config-volume
            mountPath: /fluentd/etc
          - name: pos
            mountPath: /fluentd/log
      volumes:
        - name: varlog
          hostPath:
            path: /var/log
        - name: config-volume
          configMap:
            name: fluentd-config
        - name: pos
          emptyDir: {}
```

```bash
kubectl apply -f fluentd.yaml
```

---

## 📁 Schritt 5: Zugriff auf Kibana

```bash
minikube service kibana -n logging
# oder:
kubectl port-forward svc/kibana -n logging 5601:5601
```

Dann im Browser: [http://localhost:5601](http://localhost:5601)

---

## 📁 Schritt 6: Kibana einrichten

1. Gehe zu **"Stack Management → Index Patterns"**
2. Erstelle ein Pattern: `kubernetes-*`
3. Als Zeitfeld: `@timestamp` auswählen
4. Gehe zu **"Discover"** → du solltest Logs sehen 🎉

---

## 🧹 Clean-up (optional)

```bash
kubectl delete namespace logging
```

---

## 🧠 Fazit

* **Elasticsearch** speichert Logs
* **Fluentd** sammelt Logs als DaemonSet von `/var/log/containers`
* **Kibana** zeigt sie visuell an
* Alles läuft vollständig im Cluster – **kein externer Dienst notwendig**

