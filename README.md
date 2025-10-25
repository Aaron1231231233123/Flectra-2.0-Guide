

````markdown
# Flectra 2.0 Installation Guide (Docker - Windows 10)

This document provides a step-by-step guide to installing and running **Flectra 2.0** using **Docker** on Windows 10.  
It is suitable for local development and testing environments.

---

## Prerequisites

Ensure the following are installed on your system:

- **Windows 10 (64-bit)**
- **Docker Desktop** (version 20.10 or higher)
- **Git** (optional)
- At least **4 GB of free memory**

To verify Docker installation:

```bash
docker --version
````

---

## 1. Create a Working Directory

Open **Command Prompt (CMD)** and create a project directory:

```bash
mkdir D:\Desktop\Docker\flectra-docker
cd D:\Desktop\Docker\flectra-docker
```

---

## 2. Create the Docker Compose File

Create a file named `docker-compose.yml` in the same folder and paste the following:

```yaml
version: '3.8'

services:
  db:
    image: postgres:13
    environment:
      - POSTGRES_USER=flectra
      - POSTGRES_PASSWORD=flectra
      - POSTGRES_DB=flectra
    volumes:
      - db-data:/var/lib/postgresql/data
    restart: always

  flectra:
    image: flectrahq/flectra:2.0
    depends_on:
      - db
    ports:
      - "7073:7073"
    environment:
      - HOST=db
      - PORT=5432
      - USER=flectra
      - PASSWORD=flectra
      - DATABASE=flectra
    volumes:
      - flectra-data:/var/lib/flectra
    restart: always

volumes:
  db-data:
  flectra-data:
```

---

## 3. Start the PostgreSQL Container

Start only the database container first to ensure proper initialization:

```bash
docker compose up -d db
```

Check database logs to verify readiness:

```bash
docker logs flectra-docker-db-1
```

You should see:

```
database system is ready to accept connections
```

---

## 4. Initialize the Flectra Database

Run the following command to initialize Flectra and create required tables:

```bash
docker compose run --rm flectra flectra -d flectra --db_host=db --db_port=5432 --db_user=flectra --db_password=flectra --log-level=info --init=base
```

This step ensures that core Flectra modules (including `base`, `ir.http`, and `ir_module_module`) are properly set up.

---

## 5. Start Flectra

After successful initialization, start all services:

```bash
docker compose up -d
```

Verify that both containers are running:

```bash
docker ps
```

Example output:

```
CONTAINER ID   IMAGE                   STATUS              PORTS
xxxxxx         flectrahq/flectra:2.0   Up                  0.0.0.0:7073->7073/tcp
xxxxxx         postgres:13             Up                  5432/tcp
```

---

## 6. Access the Flectra Web Interface

Open your browser and go to:

```
http://localhost:7073
```

You should now see the Flectra setup or login page.

---

## 7. Stopping and Removing Containers

To stop the containers:

```bash
docker compose down
```

To remove stored database and application data:

```bash
docker volume rm flectra-docker_flectra-data
docker volume rm flectra-docker_db-data
```

---

## 8. Troubleshooting

### Internal Server Error

Re-run the database initialization command (Step 4). The error usually occurs if Flectra starts before PostgreSQL is fully initialized.

### Port Conflict (7073)

If port 7073 is already in use, change it in `docker-compose.yml`:

```yaml
ports:
  - "8080:7073"
```

Access via `http://localhost:8080`.

### Database Connection Issues

Verify that the credentials match in both services:

```
POSTGRES_USER=flectra
POSTGRES_PASSWORD=flectra
POSTGRES_DB=flectra
```

