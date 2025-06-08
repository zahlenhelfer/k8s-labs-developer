
# 🛠️ Übung: Komplettes Java Spring Boot Multi-Stage Docker-Build Projekt

---

## 🎯 Ziel

* Komplettes Java Spring Boot Projekt mit Multi-Stage-Dockerfile
* Maven Build im Docker-Builder-Image
* Schlankes Laufzeit-Image mit JRE und fertigem Jar
* Docker-Image bauen & starten

---

## 📁 Schritt 1: Projektstruktur

- Projekt anlegen `mkdir java-spring-multistage && cd java-spring-multistage`

- Pfad anlegen `mkdir -p src/main/java/com/example/demo/`

```
java-spring-multistage/
├── Dockerfile
├── pom.xml
└── src/
    └── main/
        └── java/
            └── com/
                └── example/
                    └── demo/
                        └── DemoApplication.java
```

---

## 📁 Schritt 2: Dateien

### pom.xml

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"   
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"   
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0   
    http://maven.apache.org/xsd/maven-4.0.0.xsd">   

    <modelVersion>4.0.0</modelVersion>   

    <groupId>com.example</groupId>   
    <artifactId>demo</artifactId>   
    <version>0.0.1-SNAPSHOT</version>   
    <packaging>jar</packaging>   

    <parent>   
        <groupId>org.springframework.boot</groupId>   
        <artifactId>spring-boot-starter-parent</artifactId>   
        <version>3.0.5</version>   
        <relativePath/>   
    </parent>   

    <dependencies>   
        <dependency>   
            <groupId>org.springframework.boot</groupId>   
            <artifactId>spring-boot-starter-web</artifactId>   
        </dependency>   
    </dependencies>   

    <build>   
        <plugins>   
            <plugin>   
                <groupId>org.springframework.boot</groupId>   
                <artifactId>spring-boot-maven-plugin</artifactId>   
            </plugin>   
        </plugins>   
    </build>   

</project>
```

---

### src/main/java/com/example/demo/DemoApplication.java

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class DemoApplication {

  @GetMapping("/")
  public String hello() {
    return "Hallo von der Java Spring Boot App!";
  }

  public static void main(String[] args) {
    SpringApplication.run(DemoApplication.class, args);
  }
}
```

---

### Dockerfile

```dockerfile
# Stage 1: Build
FROM maven:3.8.7-eclipse-temurin-17 AS builder

WORKDIR /app

COPY pom.xml .
COPY src ./src

RUN mvn clean package -DskipTests

# Stage 2: Run
FROM eclipse-temurin:17-jre-alpine

WORKDIR /app

COPY --from=builder /app/target/demo-0.0.1-SNAPSHOT.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

## 📁 Schritt 3: Docker Image bauen

Im Hauptverzeichnis `java-spring-multistage`:

```bash
docker build -t java-spring-multistage .
```

---

## 📁 Schritt 4: Container starten

```bash
docker run -dit --name java-app -p 8080:8080 java-spring-multistage
```

---

## 📁 Schritt 5: Testen

```bash
curl http://localhost:8080
```

Erwarte:

```
Hallo von der Java Spring Boot App!
```

---

## 📁 Schritt 6: Aufräumen

Container mit `Ctrl+C` stoppen und optional Image löschen:

```bash
docker rm -f java-app
```

