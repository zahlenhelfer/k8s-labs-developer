
## 🛠️ Übung: Docker-Image für eine einfache Python-App erstellen

---

### 🎯 Ziel

* Eine einfache Python-Web-App (mit Flask) schreiben
* Ein Dockerfile erstellen
* Docker-Image bauen und Container starten

---

## 📁 Schritt 1: Python-App erstellen

Erstelle `app.py` mit folgendem Inhalt:

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return "Hallo von der Python-App!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

---

## 📁 Schritt 2: Requirements-Datei anlegen

Erstelle `requirements.txt`:

```
Flask==3.0.0
Werkzeug==3.0.1

```

---

## 📁 Schritt 3: Dockerfile erstellen

Erstelle eine Datei `Dockerfile`:

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]

```

---

## 📁 Schritt 4: Docker-Image bauen

```bash
docker build -t python-simple-app .
```

---

## 📁 Schritt 5: Docker-Container starten

```bash
docker run -dit --name python-app -p 5000:5000 python-simple-app
```

---

## 📁 Schritt 6: Testen

Im Browser oder per curl:

```bash
curl http://localhost:5000
```

Erwartete Ausgabe:

```
Hallo von der Python-App!
```

---

## 📁 Schritt 7: Aufräumen

Stoppe den Container mit `Ctrl+C` und lösche optional das Image:

```bash
docker container rm -f python-app
```

