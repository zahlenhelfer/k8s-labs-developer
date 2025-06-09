
## 🛠️ Übung: Security Context in einem Pod konfigurieren

### 🎯 Ziel

* Einen Pod mit Security Context anlegen
* Benutzer-ID (UID) und Gruppen-ID (GID) setzen
* Laufzeit-Einschränkungen (z.B. kein root, read-only FS) aktivieren

---

## 📁 Schritt 1: Beispiel-Pod mit Security Context

Erstelle die Datei `pod-securitycontext.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsUser: 1000          # User ID, unter der der Container läuft
    runAsGroup: 3000         # Gruppen ID
    fsGroup: 2000            # Gruppen ID für Volumes (Dateisystem)
    runAsNonRoot: true       # Container darf nicht als root laufen
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    securityContext:
      readOnlyRootFilesystem: true  # Root-Dateisystem ist schreibgeschützt
      allowPrivilegeEscalation: false  # Keine Privilege Escalation erlaubt
```

---

## 📁 Schritt 2: Pod erstellen

```bash
kubectl apply -f pod-securitycontext.yaml
```

---

## 📁 Schritt 3: Pod prüfen

```bash
kubectl get pods secure-pod
kubectl describe pod secure-pod
```

---

## 📁 Schritt 4: In den Pod rein

```bash
kubectl exec -it secure-pod -- sh
```

Überprüfe User und Rechte:

```sh
id
touch /tmp/testfile  # sollte funktionieren (tmp ist schreibbar)
touch /root/testfile # sollte **fehlschlagen** (read-only root FS)
```

---

## 🧠 Erklärung

| Feld                       | Bedeutung                                                                    |
| -------------------------- | ---------------------------------------------------------------------------- |
| `runAsUser`                | User-ID, unter der der Container läuft                                       |
| `runAsGroup`               | Gruppen-ID für den Prozess                                                   |
| `fsGroup`                  | Gruppen-ID, die auf gemounteten Volumes gesetzt wird (für Dateisystemrechte) |
| `runAsNonRoot`             | Erzwingt, dass der Container nicht als root läuft                            |
| `readOnlyRootFilesystem`   | Root-Dateisystem wird schreibgeschützt                                       |
| `allowPrivilegeEscalation` | Verbietet Privilegien-Erweiterung im Container                               |

---

## 📁 Optional: Pod ohne Security Context für Vergleich

Erstelle `pod-no-seccontext.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: insecure-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
```

Dann:

```bash
kubectl apply -f pod-no-seccontext.yaml
kubectl exec -it insecure-pod -- sh
id
```

Hier läuft der Container meist als root (UID 0).
