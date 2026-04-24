# AGENTS.md

## Project Overview

Dify is an open-source platform for developing LLM applications with an intuitive interface combining agentic AI workflows, RAG pipelines, agent capabilities, and model management.

The codebase is split into:

- **Backend API** (`/api`): Python Flask application organized with Domain-Driven Design
- **Frontend Web** (`/web`): Next.js application using TypeScript and React
- **Docker deployment** (`/docker`): Containerized deployment configurations

## Backend Workflow

- Read `api/AGENTS.md` for details
- Run backend CLI commands through `uv run --project api <command>`.
- Integration tests are CI-only and are not expected to run in the local environment.

## Frontend Workflow

- Read `web/AGENTS.md` for details

## Testing & Quality Practices

- Follow TDD: red → green → refactor.
- Use `pytest` for backend tests with Arrange-Act-Assert structure.
- Enforce strong typing; avoid `Any` and prefer explicit type annotations.
- Write self-documenting code; only add comments that explain intent.

## Language Style

- **Python**: Keep type hints on functions and attributes, and implement relevant special methods (e.g., `__repr__`, `__str__`). Prefer `TypedDict` over `dict` or `Mapping` for type safety and better code documentation.
- **TypeScript**: Use the strict config, rely on ESLint (`pnpm lint:fix` preferred) plus `pnpm type-check`, and avoid `any` types.

## General Practices

- Prefer editing existing files; add new documentation only when requested.
- Inject dependencies through constructors and preserve clean architecture boundaries.
- Handle errors with domain-specific exceptions at the correct layer.

## Server Deployment Notes

- Unless the task explicitly targets local source development, assume Dify is deployed on a server through `docker/docker-compose.yaml`.
- The server-facing HTTP entrypoint is expected to stay on port `80`. In the Docker setup this is controlled by `EXPOSE_NGINX_PORT=80` and mapped by the `nginx` service.
- Standard server startup flow:
  - `cd docker`
  - `cp .env.example .env` if `.env` does not exist yet
  - `docker compose up -d`
- After startup, the installation page is expected at `http://<server-host>/install`.
- Treat plugin-related failures such as `Failed to request plugin daemon, url: plugin/<tenant-id>/management/models` as a server-side deployment/runtime issue first, not a frontend issue.
- For that error, check these services before changing code:
  - `plugin_daemon`
  - `api`
  - `worker`
  - `db`
  - `redis`
- Relevant plugin daemon defaults in the Docker deployment:
  - internal daemon URL: `PLUGIN_DAEMON_URL=http://plugin_daemon:5002`
  - daemon container port: `5002`
  - remote debugging/install port exposure: `5003`
  - inner API target: `PLUGIN_DIFY_INNER_API_URL=http://api:5001`
- If plugin daemon requests fail on the server, inspect `docker compose ps`, `docker compose logs plugin_daemon`, and `docker compose logs api` before proposing code changes.

## Project Conventions

- Backend architecture adheres to DDD and Clean Architecture principles.
- Async work runs through Celery with Redis as the broker.
- Frontend user-facing strings must use `web/i18n/en-US/`; avoid hardcoded text.
