# Conduit Project with Docker Compose

* [Introduction](#introduction)
* [Folder structure](#folder-structure)
* [Prerequisites](#prerequisites)
* [Quickstart](#quickstart)
* [Usage](#usage)
* [Environment Variables](#environment-variables)
* [Docker Compose Services](#docker-compose-services)
* [Useful Links](#useful-links)

## Introduction

Conduit is a real‑world “Medium” clone built with Django REST Framework for the backend and Angular for the frontend. This repository provides a Dockerized setup including:

* Django backend with JWT authentication, CORS handling, and persistent SQLite storage.

* Angular frontend served by Nginx, with API proxying and CORS configuration.

* Docker Compose orchestration for easy local development and deployment.

Whether you’re exploring full‑stack patterns or looking for a reference implementation, this project has you covered.

## Folder structure

```bash
conduit-project-DA/
├── backend/                 # Django REST backend
│   ├── Dockerfile
│   ├── entrypoint.sh
│   ├── requirements.txt
│   └── conduit/             # Django project
│       ├── apps/
│       ├── core/
│       ├── settings.py
│       └── urls.py
├── frontend/                # Angular frontend
│   ├── Dockerfile
│   ├── nginx.conf
│   └── src/                 # Angular source code
├── .env.example             # Example environment variables
├── Container Checkliste.pdf # Docker Compose configuration
├── docker-compose.yaml      # Docker Compose configuration
└── README.md                # This file
```

## Prerequisites

Before you begin, ensure the following tools are installed on your system:

* Docker (>= 20.10)
* Docker Compose (>= 1.29)
* Git (to clone the repository)

## Quickstart

1. Clone GitHub Repository

    ```bash
    git clone https://github.com/DinefGH/conduit-project-DA.git
    cd conduit-project-DA
    ```

2. Copy and Configure the Environment File

    ```bash
    cp .env.example .env
    ```

    Edit .env to set your hostnames, secrets, and debug flags.

3. Build & start all services

    ```bash
    docker compose up -d --build
    ```

4. Access the application

    * Frontend: <http://localhost:8282/>
    * Backend API: <http://localhost:8001/admin/>

## Usage

1. View Logs

    ```bash
    docker compose logs -f backend
    docker compose logs -f frontend
    ```

2. Restart the Server

    ```bash
    docker compose restart
    ```

3. Persisted log files

    By using the json-file driver with rotation, Docker writes each container’s logs on the host under:

    ```swift
    /var/lib/docker/containers/<container-id>/<container-id>-json.log
    ```

4. Stop the Server

    ```bash
    docker compose down
    ```

5. Rebuild images (no cache)

    ```bash
    docker compose build --no-cache && docker compose up -d
    ```

## Environment Variables

```ini
# SECURITY
SECRET_KEY=your-very-secret-key-here
DEBUG=True
ALLOWED_HOSTS=localhost,"123.456.789.000"
DJANGO_SUPERUSER_PASSWORD=Your$tr0ngP@ssw0rd

# CORS
ALLOWED_ORIGIN=http://123.456.789.000:8282

# Backend port mapping
BACKEND_PORT=8001

# Frontend port mapping
FRONTEND_PORT=8282
```

## Docker Compose Services

* backend:

  * Builds from ./backend/Dockerfile

  * Runs the Django REST API on port 8001

  * Persists db.sqlite3 in a Docker volume

  * Entrypoint script: **`entrypoint.sh`** runs migrations and starts Gunicorn

  * JSON‑file logging with rotation

* frontend:

  * Builds from ./frontend/Dockerfile

  * Serves the Angular app via Nginx on port 8282

  * Proxies /api/ to the backend, handles CORS

  * JSON‑file logging with rotation

## Useful Links

* [Django REST Framework](https://www.django-rest-framework.org/)
* [Angular Documentation](https://angular.io/docs)
* [Docker Compose Reference](https://docs.docker.com/compose/)

---

Happy coding! 🚀
