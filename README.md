# Conduit Project with Docker Compose

* [Introduction](#introduction)
* [Folder structure](#folder-structure)
* [Prerequisites](#prerequisites)
* [Quickstart](#quickstart)
* [Usage](#usage)
* [Environment Variables](#environment-variables)
* [Docker Compose Services](#docker-compose-services)
* [CI/CD](#cicd)
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
├── .github\workflows 
│    └── deployment.yml      # GitHub Actions deployment workflow      
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

2. Initialize & update submodules (frontend & backend)

    ```bash
    git submodule update --init --recursive
    ```

3. Copy and Configure the Environment File

    ```bash
    cp example-env.txt
    ```

    Edit .env to set your hostnames, secrets, and debug flags.

4. Build & start all services

    ```bash
    docker compose up -d --build
    ```

5. Access the application

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

6. Generate SECRET_KEY

    The SECRET_KEY is the django secret key. To generate a new one see: [Stackoverflow Link](https://stackoverflow.com/questions/41298963/is-there-a-function-for-generating-settings-secret-key-in-django)

7. Monitor or Re‑run the Deployment Workflow

    * Go to your repository on GitHub and click on the Actions tab.

    * Select the Deploy‑Workflow from the left‑hand list.

    * Click on a specific run to view its logs in real time.

    * If something fails, use the Re‑run jobs dropdown to retry the deployment.

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

## CI/CD

* We’ve included a GitHub Actions workflow (.github/workflows/deployment.yml) that automatically deploys your conduit-docker branch to your remote server via SSH

    ```bash
    name: Deploy-Workflow

    on:
    push:
     branches:
         - "conduit-docker"

    jobs:
    deploy:
        name: Deploy to Remote Server
        runs-on: ubuntu-latest

        steps:
        - name: Checkout repository
            uses: actions/checkout@v4

        - name: Set up SSH key
            run: |
             mkdir -p ~/.ssh
            echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
            chmod 600 ~/.ssh/id_rsa
            # Disable host‑key checking (optional)
            echo -e "Host *\n\tStrictHostKeyChecking no\n" > ~/.ssh/config


        - name: Deploy via SSH
            uses: appleboy/ssh-action@v0.1.7
            with:
            host:     ${{ secrets.SSH_HOST }}
            username: ${{ secrets.SSH_USER }}
            key:      ${{ secrets.SSH_PRIVATE_KEY }}
            port:     ${{ secrets.SSH_PORT || '22' }}
            script: |
                set -e
                if [ -d "${{ secrets.DEPLOY_PATH }}" ]; then
                cd ${{ secrets.DEPLOY_PATH }}
                git fetch --all
                git reset --hard origin/conduit-docker
                else
                git clone git@github.com:DinefGH/conduit-project-DA.git \
                    "${{ secrets.DEPLOY_PATH }}"
                cd ${{ secrets.DEPLOY_PATH }}
                git checkout conduit-docker
                fi
                docker compose down || true
                docker compose pull
                docker compose up -d
    ```

* How to use this workflow

1. Save your secrets
    * In your GitHub repo go to Settings → Secrets → Actions and add:

    * SSH_PRIVATE_KEY – your private SSH key (PEM format)

    * SSH_HOST – your server’s IP or hostname

    * SSH_USER – the SSH user (e.g. ubuntu)

    * SSH_PORT – (optional, default 22)

    * DEPLOY_PATH – absolute path on the server where Conduit lives

2. Push to the conduit-docker branch
    * On every push the workflow will run and deploy your updated stack.

3. Monitor runs
    * Check Actions in GitHub to see live logs of the deployment step.

## Useful Links

* [Django REST Framework](https://www.django-rest-framework.org/)
* [Angular Documentation](https://angular.io/docs)
* [Docker Compose Reference](https://docs.docker.com/compose/)

---

Happy coding! 🚀
