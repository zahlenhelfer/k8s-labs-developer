
# 🛠️ Übung: Docker-Image für eine einfache Node.js-App erstellen

---

## 🎯 Ziel

* Eine kleine Node.js-Web-App schreiben (mit Express)
* Ein Dockerfile erstellen
* Docker-Image bauen und Container starten

---

## 📁 Schritt 1: Node.js-App erstellen

Lege im neuen Ordner folgende Dateien an:

### `package.json`

```json
{
  "name": "node-simple-app",
  "version": "1.0.0",
  "description": "Einfache Node.js App mit Express",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
```

### `app.js`

```js
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.send('Hallo von der Node.js-App!');
});

app.listen(port, () => {
  console.log(`Server läuft auf http://localhost:${port}`);
});
```

---

## 📁 Schritt 2: Dockerfile erstellen

Erstelle `Dockerfile`:

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package.json package-lock.json* ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

---

## 📁 Schritt 3: Docker-Image bauen

```bash
docker build -t node-simple-app .
```

---

## 📁 Schritt 4: Container starten

```bash
docker run -dit --name node-app -p 3000:3000 node-simple-app
```

---

## 📁 Schritt 5: Testen

Im Browser oder per curl:

```bash
curl http://localhost:3000
```

Erwarte Ausgabe:

```
Hallo von der Node.js-App!
```

---

## 📁 Schritt 6: Aufräumen

```bash
docker container rm -f node-app
```
