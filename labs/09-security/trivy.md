
# 🛠️ Übung: Trivy installieren und nginx Docker-Image scannen

---

## 🎯 Ziel

* Trivy installieren
* Docker-Image `nginx` herunterladen
* Image mit Trivy auf Schwachstellen prüfen

---

## 📁 Schritt 1: Trivy installieren

### Linux (Debian/Ubuntu)

```bash
sudo apt-get install -y wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy
```

### Alternativ: Trivy als Binary herunterladen

[https://github.com/aquasecurity/trivy/releases](https://github.com/aquasecurity/trivy/releases)

---

## 📁 Schritt 2: Docker-Image herunterladen

```bash
docker pull nginx
```

---

## 📁 Schritt 3: Image mit Trivy scannen

```bash
trivy image nginx
```

Du bekommst eine Liste von gefundenen Schwachstellen mit Details zu Schweregrad, CVE-ID, Beschreibung.

---

## 📁 Schritt 4: Optionale Parameter

* Nur Schwachstellen ab hohem Schweregrad sehen:

```bash
trivy image --severity HIGH,CRITICAL nginx
```

* Nur Ergebnis im JSON-Format ausgeben:

```bash
trivy image --format json nginx
```

