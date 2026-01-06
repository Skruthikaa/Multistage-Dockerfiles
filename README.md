# Docker Multi-Stage Build Documentation

This document explains three separate **multi-stage Dockerfiles** used to build and deploy applications from GitHub repositories. Each Dockerfile follows best practices for **smaller images, security, and maintainability**.

---

## 1. Java Web Application (Maven + Tomcat)

**Repository:** `Skruthikaa/Rock-paper-web-`

### Dockerfile Used

```dockerfile
############################
# 1. CLONE STAGE
############################
FROM alpine/git AS clone
WORKDIR /app
RUN git clone https://github.com/Skruthikaa/Rock-paper-web-.git

############################
# 2. BUILD STAGE (MAVEN)
############################
FROM maven:3.9.6-eclipse-temurin-17 AS build
WORKDIR /app
COPY --from=clone /app/Rock-paper-web- /app
RUN mvn clean package -DskipTests

############################
# 3. DEPLOY STAGE (TOMCAT)
############################
FROM tomcat:10.1-jdk17
WORKDIR /usr/local/tomcat/webapps
COPY --from=build /app/target/*.war ROOT.war
EXPOSE 8080
CMD ["catalina.sh", "run"]
```

### Step-by-Step Explanation

**Clone Stage**

* `alpine/git` is used because it is lightweight.
* The GitHub repository is cloned into `/app`.

**Build Stage (Maven)**

* Uses Maven with Java 17 to compile the project.
* `mvn clean package -DskipTests` builds the WAR file while skipping tests for faster builds.
* The output WAR file is generated in the `/target` directory.

**Deploy Stage (Tomcat)**

* Apache Tomcat is used as the runtime server.
* The generated WAR file is copied as `ROOT.war` so it runs as the default application.
* Port `8080` is exposed to access the web app.

### Purpose

Build a Java web application using **Maven** and deploy it on **Apache Tomcat** as a WAR file.

### How to Build & Run

```bash
docker build -t rock-paper-app .
docker run -p 8080:8080 rock-paper-app
```

Access: `http://localhost:8080`

---

## 2. Frontend Application (Node.js + Nginx)

**Repository:** `Skruthikaa/pomodoro-app-js`

### Dockerfile Used

```dockerfile
######################################
# 1. CLONE STAGE
######################################
FROM alpine/git AS clone
WORKDIR /app
RUN git clone https://github.com/Skruthikaa/pomodoro-app-js.git

######################################
# 2. BUILD STAGE (NODE)
######################################
FROM node:18-alpine AS build
WORKDIR /app
COPY --from=clone /app/pomodoro-app-js /app
RUN npm install
RUN npm run build

######################################
# 3. DEPLOY STAGE (NGINX)
######################################
FROM nginx:alpine
RUN rm -rf /usr/share/nginx/html/*
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Step-by-Step Explanation

**Clone Stage**

* Uses `alpine/git` to clone the frontend source code.

**Build Stage (Node.js)**

* Uses Node.js 18 on Alpine Linux.
* `npm install` installs all dependencies.
* `npm run build` creates optimized static files inside the `/dist` folder.

**Deploy Stage (Nginx)**

* Nginx is used to serve static frontend files.
* Default HTML content is removed for a clean deployment.
* The built files from `/dist` are copied to Nginx’s web directory.
* Port `80` is exposed for browser access.

### Purpose

Build a JavaScript frontend app and serve static files using **Nginx**.

### How to Build & Run

```bash
docker build -t pomodoro-app .
docker run -p 80:80 pomodoro-app
```

Access: `http://localhost`

---

## 3. Python Application

**Repository:** `Skruthikaa/Python`

### Dockerfile Used

```dockerfile
######################################
# 1. CLONE STAGE
######################################
FROM alpine/git AS clone
WORKDIR /app
RUN git clone https://github.com/Skruthikaa/Python.git

######################################
# 2. BUILD STAGE
######################################
FROM python:3.11-slim AS build
WORKDIR /app
COPY --from=clone /app/Python /app

######################################
# 3. RUN STAGE
######################################
FROM python:3.11-slim
WORKDIR /app
COPY --from=build /app /app
CMD ["python", "game.py"]
```

### Step-by-Step Explanation

**Clone Stage**

* Uses `alpine/git` to clone the Python repository.

**Build Stage**

* Uses a slim Python image to keep the image size small.
* Copies application files into the container.
* No dependency installation is required as only standard libraries are used.

**Run Stage**

* The final image runs the Python application.
* `game.py` is executed when the container starts.

### Purpose

Run a Python-based game or script inside a Docker container.

### How to Build & Run

```bash
docker build -t python-game .
docker run python-game
```

---

## Key Benefits of Multi-Stage Builds

* Smaller final image size
* Improved security
* Faster deployments
* Clear separation of responsibilities

---



