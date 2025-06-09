
## 🛠️ Übung: Pod Security Standards in Kubernetes aktivieren und testen

---

### 🎯 Ziel

* Ein Namespace mit Pod Security Standards Annotation anlegen
* Pods mit verschiedenen Sicherheitsleveln (Privileged, Baseline, Restricted) testen
* Verstehen, wie Kubernetes Pod Security Admission diese Pods zulässt oder ablehnt

---

## 📁 Schritt 1: Namespace mit Pod Security Standards Annotation anlegen

Erstelle `namespace-sec.yaml`:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: secure-ns
  annotations:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/enforce-version: latest
```

Hier gilt der Modus **`restricted`**, das ist der strengste Standard.

```bash
kubectl apply -f namespace-sec.yaml
```

---

## 📁 Schritt 2: Pod mit unsicheren Einstellungen erstellen (soll abgelehnt werden)

Erstelle `pod-privileged.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: privileged-pod
  namespace: secure-ns
spec:
  containers:
  - name: busybox
    image: busybox
    securityContext:
      privileged: true
    command: ["sh", "-c", "sleep 3600"]
```

Versuche den Pod zu erstellen:

```bash
kubectl apply -f pod-privileged.yaml
```

**Erwartung:** Der Pod wird **abgelehnt**, weil `privileged: true` nicht im `restricted` Standard erlaubt ist.

---

## 📁 Schritt 3: Pod mit Baseline-konformen Einstellungen erstellen (soll akzeptiert werden)

Erstelle `pod-baseline.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: baseline-pod
  namespace: secure-ns
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
```

```bash
kubectl apply -f pod-baseline.yaml
```

Der Pod sollte akzeptiert und gestartet werden.

---

## 📁 Schritt 4: Pod ohne SecurityContext erstellen (wird im restricted Namespace oft abgelehnt)

Erstelle `pod-no-sec.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: no-sec-pod
  namespace: secure-ns
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
```

```bash
kubectl apply -f pod-no-sec.yaml
```

Dieser Pod wird **wahrscheinlich abgelehnt** (wegen fehlendem `runAsNonRoot`).

---

## 📁 Schritt 5: Pod Security Standards in verschiedenen Modi

Du kannst die Annotation anpassen:

| Modus        | Beschreibung                                       |
| ------------ | -------------------------------------------------- |
| `privileged` | Keine Einschränkungen (unsicher)                   |
| `baseline`   | Basis-Sicherheitsregeln (realistischer Kompromiss) |
| `restricted` | Strengste Sicherheitsregeln                        |

Beispiel für `baseline`:

```yaml
annotations:
  pod-security.kubernetes.io/enforce: baseline
  pod-security.kubernetes.io/enforce-version: latest
```

---

## 📁 Schritt 6: Namespace ändern und testen

```bash
kubectl delete ns secure-ns
kubectl apply -f namespace-sec.yaml  # mit anderen Annotationen
```

Dann die Pods erneut testen.

---

## 🧠 Erklärung

| Begriff                          | Beschreibung                                                         |
| -------------------------------- | -------------------------------------------------------------------- |
| Pod Security Standards (PSS)     | Eingebaute Kubernetes Admission Controller Regeln für Pod-Sicherheit |
| enforce                          | Erzwingt den angegebenen Sicherheitsstandard in einem Namespace      |
| privileged, baseline, restricted | Verschiedene Sicherheitsprofile mit unterschiedlicher Strenge        |
| Admission Controller             | Blockiert Pods, die nicht den Regeln entsprechen                     |

