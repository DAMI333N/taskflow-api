# TaskFlow API

A small Spring Boot REST API for managing tasks: list, read, create, update,
delete, mark as complete, and a summary of what is
done and what is pending. Tasks live in memory and the code is split into the
controller, service, repository and model layers taught in class.

Built for NTU AI Engineering, Module 3: Backend Development (Java and Spring
Boot). Its data shape matches the Module 2 frontend,
[task-manager](https://github.com/DAMI333N/task-manager), so the two can be
wired together later without changing either model.

## Features

- `GET /api/tasks` returns every task as a bare JSON array
- `GET /api/tasks/{id}`, `POST /api/tasks`, `PUT /api/tasks/{id}` and
  `DELETE /api/tasks/{id}` for the usual CRUD, with 201 on create and 200
  everywhere else
- `PUT /api/tasks/{id}/complete` marks a task as done
- `GET /api/tasks/summary` asks a language model for a short summary of the
  task list (Task 6, Spring AI)
- Six seed tasks preloaded at startup, the same six the frontend ships with
- Consistent JSON error bodies: 404 for an unknown id, 400 for a blank title,
  an unknown status or priority, a non-numeric id or malformed JSON, 502 when
  the model cannot be reached
- CORS enabled for the frontend's Vite dev server

## Tech stack

- Java 21, Maven
- Spring Boot 4.1.1 with Spring Web (Spring MVC), Lombok and DevTools
- Spring AI 2.0.1 with the OpenAI starter, pointed at OpenRouter
- JUnit 5, Mockito and MockMvc for tests, no database

## Getting started

```bash
git clone https://github.com/DAMI333N/module3-taskflow-api
cd module3-taskflow-api
mvn spring-boot:run
```

The API listens on `http://localhost:8080`. Tasks 1 to 5 work with no
configuration at all. The summary endpoint needs an API key, which the app reads
from an environment variable and which never appears in the project:

```bash
echo 'export OPENAI_API_KEY=your-real-key-here' >> ~/.bashrc
source ~/.bashrc
```

`application.properties` refers to it as `spring.ai.openai.api-key=${OPENAI_API_KEY}`
and points the OpenAI-compatible client at OpenRouter
(`spring.ai.openai.base-url=https://openrouter.ai/api/v1`, model `openrouter/free`),
so the variable holds an OpenRouter key. Without it the app still starts and only
`GET /api/tasks/summary` fails, with a 502.

Other commands:

```bash
mvn test                  # 39 unit, web-layer and integration tests
mvn clean package         # build target/taskflow-api-1.0.0.jar
```

A Postman collection covering every endpoint is in
`postman/TaskFlow.postman_collection.json`; import it and set `baseUrl` if the
API is not on `localhost:8080`.

## Project structure

```
src/main/java/sg/edu/ntu/taskflowapi/
  TaskflowApiApplication.java   Spring Boot entry point
  controller/TaskController     the seven endpoints, ResponseEntity + HttpStatus
  service/TaskService           business rules: validation, status/completed sync
  service/TaskSummaryService    Task 6: builds the prompt and calls the model
  repository/TaskRepository     ArrayList storage, counter ids, seed data
  model/Task                    plain POJO, Lombok getters and setters
  exceptions/                   TaskNotFoundException, InvalidTaskException,
                                SummaryUnavailableException, ErrorResponse,
                                GlobalExceptionHandler (@RestControllerAdvice)
  config/WebConfig              CORS for the frontend origin
src/test/java/sg/edu/ntu/taskflowapi/
  service/TaskServiceTest       Mockito unit tests
  controller/TaskControllerTest @WebMvcTest with the services mocked
  TaskFlowIntegrationTest       @SpringBootTest + MockMvc through the real stack
```

## API

| Method | Path | What it does | Status |
|---|---|---|---|
| GET | `/api/tasks` | Get all tasks | 200 |
| GET | `/api/tasks/{id}` | Get one task by id | 200, 404 |
| POST | `/api/tasks` | Create a new task | 201, 400 |
| PUT | `/api/tasks/{id}` | Replace an existing task | 200, 400, 404 |
| DELETE | `/api/tasks/{id}` | Delete a task (empty body) | 200, 404 |
| PUT | `/api/tasks/{id}/complete` | Mark a task as complete | 200, 404 |
| GET | `/api/tasks/summary` | Plain-text AI summary of all tasks | 200, 502 |

A task looks like this:

```json
{
  "id": 3,
  "title": "Build login page",
  "description": "Create a login form with email and password fields and basic validation.",
  "status": "in-progress",
  "priority": "high",
  "completed": false
}
```

`title` is required. `status` is one of `todo`, `in-progress` or `done` and
defaults to `todo`; `priority` is one of `low`, `medium` or `high` and defaults
to `medium`; `description` is optional. `id` is assigned by the server and any
id sent in a request body is ignored.

Create a task with just the fields you have:

```bash
curl -X POST localhost:8080/api/tasks \
  -H 'Content-Type: application/json' \
  -d '{"title": "Write the README", "priority": "low"}'
```

### status and completed

The assignment asks for a `completed` flag; the frontend tracks a three-state
`status`. The API carries both and keeps them consistent. `status` is the
source of truth: `done` means `completed` is `true`, anything else means
`false`, whatever the request body said. When a request leaves `status` out,
it is derived from `completed`, so `{"title": "x", "completed": true}` creates
a `done` task. `PUT /api/tasks/{id}/complete` sets both.

`PUT` replaces the whole task: fields left out of the body fall back to the
defaults above rather than keeping their previous values.

### Errors

Every error the API reports itself has the same body:

```json
{
  "status": 404,
  "message": "Could not find task with id: 42",
  "timestamp": "2026-09-14T01:06:07.0007"
}
```

## AI summary (Task 6)

`GET /api/tasks/summary` renders the task list as text, sends it to the model
with a system prompt asking for one short paragraph (done first, then pending,
high-priority items by name, no bullet points) and returns the reply as
`text/plain`. Task text is injected through Spring AI's `.param()` rather than
string concatenation, so a title containing braces cannot break the prompt
template. With no tasks the endpoint answers without calling the model.

The `ChatClient` is built from the auto-configured `ChatClient.Builder` inside
`TaskSummaryService`, which is kept separate from `TaskService` so the CRUD
layer has no dependency on Spring AI.

## Using it from the task-manager frontend

The frontend currently keeps its tasks in `localStorage`. Replacing that with
this API needs no change to its data model:

- `GET http://localhost:8080/api/tasks` returns the same bare array shape the
  frontend already stores, with the same `status` and `priority` strings
- ids are numbers; the frontend already compares ids with `String()` on both
  sides, so nothing changes there
- `addTask` maps to `POST /api/tasks` with `{title, description, status, priority}`
- `updateTask` maps to `PUT /api/tasks/{id}` with the full task object, exactly
  what the detail page already assembles
- `deleteTask` maps to `DELETE /api/tasks/{id}`
- marking a task done is either a `PUT` with `status: "done"` or
  `PUT /api/tasks/{id}/complete`
- CORS is open for `http://localhost:5173` and `http://127.0.0.1:5173`; add
  other origins to `taskflow.cors.allowed-origins` in `application.properties`

Drag-and-drop ordering stays a frontend concern; the API returns tasks in
creation order.

## CI pipeline (Option 1)

GitHub Actions runs [`.github/workflows/ci.yml`](.github/workflows/ci.yml) on
every push. The test job runs `./mvnw -B verify` on Temurin 21. On `main` only,
a second job builds the image from the Dockerfile and pushes it to Docker Hub
tagged `main` and `sha-<short commit>`. Docker Hub credentials are the
`DOCKER_USERNAME` and `DOCKER_TOKEN` repository secrets, never in the code.

- Successful pipeline run: [run 39737766680](https://github.com/DAMI333N/taskflow-api/actions/runs/39737766680)
  (39 tests, image pushed as `main` and `sha-0a2cfc7`)
- Image on Docker Hub: [dami333n/taskflow-api](https://hub.docker.com/r/dami333n/taskflow-api)

