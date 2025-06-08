
## 🛠️ Übung: Docker-Image für eine einfache Go-App erstellen

---

### 🎯 Ziel

* Eine kleine Go-Anwendung schreiben
* Ein Dockerfile für diese App erstellen
* Ein Docker-Image bauen und lokal testen

---

## 📁 Schritt 1: Go-App schreiben

- Erstelle eine Datei `main.go` mit folgendem Inhalt:

```go
package main

import (
  "fmt"
  "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
  fmt.Fprintf(w, "Hallo von der Go-App!")
}

func main() {
  http.HandleFunc("/", handler)
  fmt.Println("Starte Server auf :8080")
  http.ListenAndServe(":8080", nil)
}
```

- Erstelle eine Datei `go.mod` mit folgendem Inhalt:
```go
module example.com/simple-go-app

go 1.22
```

---

## 📁 Schritt 2: Dockerfile erstellen

Erstelle eine Datei `Dockerfile` im selben Verzeichnis:

```dockerfile
# Start from the official Go image as the build stage
FROM golang:1.22 AS builder

# Set the working directory inside the container
WORKDIR /app

# Copy go mod and sum files
COPY go.mod ./

# Download dependencies
RUN go mod download

# Copy the source code
COPY . .

# Build the Go app
RUN CGO_ENABLED=0 go build -o main .

# Use a minimal base image (distroless or alpine) for the final container
FROM debian:bullseye-slim

# Set working directory in the new container
WORKDIR /root/

# Copy the binary from the builder stage
COPY --from=builder /app/main .

# Expose the application port
EXPOSE 8080

# Command to run the executable
CMD ["./main"]
```

---

## 📁 Schritt 3: Docker Image bauen

Im Verzeichnis mit `main.go` und `Dockerfile`:

```bash
docker build -t go-simple-app .
```

---

## 📁 Schritt 4: Docker Container starten

```bash
docker run -dit --name go-app -p 8080:8080 go-simple-app
```

---

## 📁 Schritt 5: Test im Browser oder mit curl

Öffne im Browser oder mit curl:

```bash
curl http://localhost:8080
```

Antwort sollte sein:

```
Hallo von der Go-App!
```

---

## 📁 Schritt 6: Aufräumen

Stoppe den Container:

```bash
docker container rm -f go-app
```

