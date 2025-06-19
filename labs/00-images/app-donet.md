
# 🛠️ Übung: Multi-Stage Docker-Image für eine dotNET Core Web API erstellen
---

## 🎯 Ziel

* Eine einfache .NET Core Web API schreiben
* Multi-Stage Dockerfile nutzen (Build + Runtime)
* Image bauen und Container starten

---
## 📁 Schritt 0: .NET Core-Installation

```bash
sudo apt update && sudo apt install -y wget apt-transport-https
wget https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
sudo apt update
sudo apt install -y dotnet-sdk-8.0
```

---

## 📁 Schritt 1: Projekt anlegen

Im Terminal:

```bash
dotnet new console -n MySimpleDotnetApp
cd MySimpleDotnetApp
```

---

## 📁 Schritt 2: 🧠 `Program.cs` anpassen:

```csharp
using System;

namespace MySimpleDotnetApp
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello, World!");
        }
    }
}

```

---

## 📁 Schritt 3: Multi-Stage Dockerfile erstellen

Lege im Projektordner eine Datei `Dockerfile` an:

```dockerfile
# Use the official .NET SDK image to build the application
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["MySimpleDotnetApp.csproj", "."]
RUN dotnet restore "MySimpleDotnetApp.csproj"
COPY . .
RUN dotnet build "MySimpleDotnetApp.csproj" -c Release -o /app/build

# Publish the application
FROM build AS publish
RUN dotnet publish "MySimpleDotnetApp.csproj" -c Release -o /app/publish

# Use the ASP.NET Core runtime image to run the application
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "MySimpleDotnetApp.dll"]

```

---

## 📁 Schritt 4: Docker Image bauen

```bash
docker build -t mysimpledotnetapp .
```

---

## 📁 Schritt 5: Container starten

```bash
docker run 
docker run --name dotnet-app mysimpledotnetapp
```

---

## 📁 Schritt 6: Aufräumen

Container stoppen (Ctrl+C) und Image löschen:

```bash
docker container rm -f dotnet-app
```

