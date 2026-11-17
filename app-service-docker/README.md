# Spring Boot Microservice Containerization using Docker

A simple Spring Boot REST microservice (`app-service`) that you package as a JAR and run inside a Docker container.

| Item | Value |
|---|---|
| Group ID / Artifact ID | `com.example` / `app-service` |
| Java | 17 |
| Spring Boot | 3.5.7 |
| Build tool | Maven (JAR) |
| Port | 8080 |
| Endpoint | `GET /api/data` |

---

## 1. Concepts (read this first)

**Containerization** means packing an application together with everything it needs to run (Java runtime, libraries, settings) into one unit. It then runs the same way on any computer that has a container engine, which removes "it works on my machine" problems.

**Docker** is the most popular tool for building and running containers.

**Docker image** is a read-only template: your app + Java 17 + settings. Think of it as a *recipe / installer*. Here it is named `app-service:v1`.

**Docker container** is a running instance of an image. Think of it as the *running program*. You can start many containers from one image.

**Dockerfile** is a text file with instructions that tell Docker how to build an image.

---

## 2. Prerequisites (Windows)

- Spring Tools for Eclipse (STS) with Java 17 JDK configured
- Docker Desktop for Windows, installed and **running** (whale icon in the system tray shows "Docker Desktop is running")
- Internet connection (Maven and Docker download dependencies the first time)

---

## 3. Import the project into STS

1. Unzip `app-service-docker.zip` to a simple path, e.g. `C:\practicals\app-service-docker`.
2. In STS: **File > Import... > Maven > Existing Maven Projects > Next**.
3. Click **Browse...** and select the `app-service-docker` folder (the one containing `pom.xml`).
4. Make sure `/pom.xml` is ticked, then click **Finish**. Wait for the Maven dependency download to finish (progress at the bottom right).
5. Check the JDK: right-click the project > **Properties > Java Compiler** should say 17.

(Optional) Run it without Docker first: right-click `AppServiceApplication.java` > **Run As > Spring Boot App**, open `http://localhost:8080/api/data`, then stop it with the red square in the Console. **Stop it before running the container**, otherwise port 8080 will be busy.

---

## 4. Package the application as a JAR

In STS: right-click the project > **Run As > Maven build...**
- **Goals:** `clean package`
- Click **Run**.

Or from a terminal in the project root folder:

```
mvn clean package
```

(Use `.\mvnw` only if you added the Maven wrapper; this project does not include it, so use the STS option or an installed Maven.)

When you see `BUILD SUCCESS`, the file **`target/app-service.jar`** exists. If you don't see `target` in STS, right-click the project > **Refresh (F5)**.

> The Dockerfile copies exactly `target/app-service.jar`. This name is fixed by `<finalName>app-service</finalName>` in `pom.xml`.

---

## STEP 1: Write the Dockerfile

The `Dockerfile` is already in the project root folder:

```dockerfile
FROM eclipse-temurin:17-jre
WORKDIR /app
COPY target/app-service.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

What each line does:

| Line | Meaning |
|---|---|
| `FROM eclipse-temurin:17-jre` | Start from an image that already contains a Java 17 runtime |
| `WORKDIR /app` | Work inside the `/app` folder in the container |
| `COPY target/app-service.jar app.jar` | Copy your JAR into the image as `/app/app.jar` |
| `EXPOSE 8080` | Documents that the app listens on port 8080 |
| `ENTRYPOINT [...]` | The command that runs when the container starts |

The `.dockerignore` file keeps unneeded files (like `src`, `.git`) out of the build, but keeps `target/app-service.jar`.

---

## STEP 2: Build the Docker image

1. **Start Docker Desktop** and wait until it says it is running.
2. **Open a terminal in the project root folder** (the folder containing `Dockerfile` and `pom.xml`). Easiest ways:
   - In **File Explorer**, open the `app-service-docker` folder, click the address bar, type `cmd`, press **Enter**. The terminal opens already in the right folder.
   - Or open **Command Prompt / PowerShell** and navigate:
     ```
     cd C:\practicals\app-service-docker
     ```
     (replace the path with where you unzipped it). Check with `dir`: you should see `Dockerfile`, `pom.xml`, `src`, `target`.
3. Run the command (don't forget the dot at the end; it means "current folder"):

```
docker build -t app-service:v1 .
```

4. Verify the image was created:

```
docker images
```

You should see a line like:

```
REPOSITORY    TAG   IMAGE ID       CREATED          SIZE
app-service   v1    xxxxxxxxxxxx   a few seconds    ~300MB
```

---

## STEP 3: Run and verify the container

1. Start the container (foreground):

```
docker run -p 8080:8080 app-service:v1
```

`-p 8080:8080` maps port 8080 of your Windows machine to port 8080 inside the container. Wait for a log line like `Started AppServiceApplication in ... seconds`.

2. **Check running containers.** Open a **second** terminal and run:

```
docker ps
```

You should see `app-service:v1` with `0.0.0.0:8080->8080/tcp`.

3. **View logs:**
   - In the foreground, logs appear directly in the terminal where you ran `docker run`.
   - From another terminal: `docker logs <container-id-or-name>` (add `-f` to follow live logs).

4. **Test in the browser:** open

```
http://localhost:8080/api/data
```

Expected response (HTTP 200 OK):

```
Microservice is running successfully inside Docker!
```

To see the status code too, run `curl -i http://localhost:8080/api/data` in a terminal (you should see `HTTP/1.1 200`).

5. **Stop the container:**
   - Foreground: click the terminal running it and press **Ctrl + C**.
   - Detached mode (background):
     ```
     docker run -d -p 8080:8080 --name app-service-container app-service:v1
     docker ps
     docker logs app-service-container
     docker stop app-service-container
     ```
   - Optionally remove stopped containers: `docker rm app-service-container`

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `Cannot connect to the Docker daemon` / `error during connect` | Docker Desktop is not running. Start it and wait until it is ready. |
| `COPY failed: file not found ... target/app-service.jar` | Run `mvn clean package` first (or STS **Run As > Maven build > clean package**) and refresh the project. |
| `Bind for 0.0.0.0:8080 failed: port is already allocated` / `Port 8080 was already in use` | Stop the STS-run app or another container using 8080 (`docker ps`, then `docker stop <id>`). |
| `docker build` says `"docker build" requires exactly 1 argument` | You missed the `.` at the end of the command. |
| Browser can't connect | Make sure the container is still running (`docker ps`) and you used `-p 8080:8080`. |
| Wrong Java version error in STS | Project Properties > Java Compiler = 17, and use a JDK 17 in **Window > Preferences > Java > Installed JREs**. |

## Project structure

```
app-service-docker/
├── pom.xml
├── Dockerfile
├── .dockerignore
├── README.md
└── src/main/
    ├── java/com/example/appservice/
    │   ├── AppServiceApplication.java
    │   └── controller/DataController.java
    └── resources/application.properties
```
