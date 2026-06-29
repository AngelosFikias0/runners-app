# Runners App

A RESTful API for tracking athletic runs, built with Spring Boot 3, PostgreSQL, and Docker.

---

## Architecture

```
Client (HTTP)
    │
    ▼
RunController          ← request mapping, validation, HTTP status
    │
    ▼
RunRepository          ← data access interface
    │
    ▼
JdbcRunRepository      ← Spring JdbcClient implementation
    │
    ▼
H2 (dev) / PostgreSQL (prod)
```

Cross-cutting concerns (validation errors, illegal arguments) are handled centrally by `GlobalExceptionHandler` using RFC 9457 `ProblemDetail` responses.

---

## API Endpoints

| Method   | Endpoint                     | Status  | Description              |
| -------- | ---------------------------- | ------- | ------------------------ |
| `GET`    | `/api/runs`                  | 200     | List all runs            |
| `GET`    | `/api/runs/{id}`             | 200/404 | Get a run by ID          |
| `GET`    | `/api/runs/search?location=` | 200     | Filter runs by location  |
| `POST`   | `/api/runs`                  | 201     | Create a new run         |
| `PUT`    | `/api/runs/{id}`             | 204     | Update an existing run   |
| `DELETE` | `/api/runs/{id}`             | 204     | Delete a run             |
| `GET`    | `/actuator/health`           | 200     | Application health check |
| `GET`    | `/actuator/metrics`          | 200     | Application metrics      |

### Sample Request Body (POST / PUT)

```json
{
  "id": 11,
  "title": "Evening Trail Run",
  "startedOn": "2024-05-01T18:00:00",
  "completedOn": "2024-05-01T19:30:00",
  "miles": 8,
  "location": "OUTDOOR"
}
```

`location` must be `INDOOR` or `OUTDOOR`.

---

## Tech Stack

| Layer         | Technology                      |
| ------------- | ------------------------------- |
| Framework     | Spring Boot 3.2                 |
| Language      | Java 21                         |
| Build         | Maven (Maven Wrapper)           |
| Database      | H2 (dev) · PostgreSQL 16 (prod) |
| Data access   | Spring JdbcClient               |
| Observability | Spring Boot Actuator            |
| Container     | Docker + Docker Compose         |
| CI            | GitHub Actions                  |

---

## Local Development (H2)

**Prerequisites:** Java 21, Maven 3.8+

```bash
cd Runners_Application
./mvnw spring-boot:run
```

The app starts on `http://localhost:8080`.
H2 console: `http://localhost:8080/h2-console` · JDBC URL: `jdbc:h2:mem:runnerz` · User: `sa`

10 sample runs are loaded automatically from `src/main/resources/data/runs.json` on first start.

---

## Docker (PostgreSQL)

**Prerequisites:** Docker, Docker Compose

```bash
# Build and start both services
docker compose up --build

# Stop and remove containers
docker compose down

# Stop and remove containers + database volume
docker compose down -v
```

The app will be available at `http://localhost:8080` backed by a persistent PostgreSQL instance.

### Environment Variables

| Variable                 | Default     | Description                 |
| ------------------------ | ----------- | --------------------------- |
| `SPRING_PROFILES_ACTIVE` | `prod`      | Activates PostgreSQL config |
| `DB_HOST`                | `localhost` | PostgreSQL host             |
| `DB_PORT`                | `5432`      | PostgreSQL port             |
| `DB_NAME`                | `runnerz`   | Database name               |
| `DB_USER`                | `postgres`  | Database username           |
| `DB_PASS`                | `postgres`  | Database password           |

Override any variable in `docker-compose.yml` or pass them via `-e` flags.

---

## CI / CD

GitHub Actions runs on every push and pull request to `main`:

1. **build-and-test** — compiles the project and runs all tests (`./mvnw clean verify`)
2. **docker** — builds the Docker image on merges to `main` (tags with commit SHA + `latest`)

See `.github/workflows/ci.yml`.

---

## Project Structure

```
runners-app/
├── .github/workflows/ci.yml          ← GitHub Actions pipeline
├── docs/spring-reference.md          ← Spring Boot reference notes
├── docker-compose.yml                ← App + PostgreSQL services
└── Runners_Application/
    ├── Dockerfile                    ← Multi-stage build
    ├── pom.xml
    └── src/main/
        ├── java/dev/fikias/runners/
        │   ├── Application.java
        │   ├── GlobalExceptionHandler.java
        │   └── run/
        │       ├── Run.java                 ← record with validation
        │       ├── RunController.java       ← REST layer
        │       ├── RunRepository.java       ← interface
        │       ├── JdbcRunRepository.java   ← JDBC implementation
        │       ├── RunJsonDataLoader.java   ← seeds DB on first start
        │       ├── RunNotFoundException.java
        │       ├── Runs.java
        │       └── Location.java
        └── resources/
            ├── application.properties       ← dev (H2)
            ├── application-prod.properties  ← prod (PostgreSQL)
            ├── schema.sql
            └── data/runs.json
```
