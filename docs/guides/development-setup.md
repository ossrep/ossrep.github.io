# Development Setup

Detailed instructions for setting up your development environment for OSSREP.

## Java Development

### Install JDK 21

=== "macOS (Homebrew)"

    ```bash
    brew install openjdk@21
    ```

=== "Ubuntu/Debian"

    ```bash
    sudo apt install openjdk-21-jdk
    ```

=== "SDKMAN (Recommended)"

    ```bash
    sdk install java 21-tem
    ```

### Install Maven

Maven 3.9+ is required for building Quarkus services.

=== "macOS (Homebrew)"

    ```bash
    brew install maven
    ```

=== "SDKMAN"

    ```bash
    sdk install maven
    ```

### IDE Setup

#### IntelliJ IDEA (Recommended)

1. Install the Quarkus plugin
2. Enable annotation processing
3. Import projects as Maven projects

#### VS Code

1. Install "Extension Pack for Java"
2. Install "Quarkus" extension

---

## Frontend Development

### Install Node.js

Node.js 20+ is required for Angular development.

=== "macOS (Homebrew)"

    ```bash
    brew install node@20
    ```

=== "nvm (Recommended)"

    ```bash
    nvm install 20
    nvm use 20
    ```

### Install Angular CLI

```bash
npm install -g @angular/cli
```

---

## Docker Setup

### Install Docker

Follow the official Docker installation guide for your operating system:

- [Docker Desktop for Mac](https://docs.docker.com/desktop/install/mac-install/)
- [Docker Desktop for Windows](https://docs.docker.com/desktop/install/windows-install/)
- [Docker Engine for Linux](https://docs.docker.com/engine/install/)

### Local Infrastructure

Create a `docker-compose.yml` in your workspace root:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_USER: ossrep
      POSTGRES_PASSWORD: ossrep
      POSTGRES_DB: ossrep
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  # Add other services as needed (Kafka, Redis, etc.)

volumes:
  postgres_data:
```

Start the infrastructure:

```bash
docker-compose up -d
```

---

## Database Setup

### Connect to PostgreSQL

```bash
psql -h localhost -U ossrep -d ossrep
```

### Create service databases

Each microservice should have its own database:

```sql
CREATE DATABASE customer_service;
CREATE DATABASE billing_service;
CREATE DATABASE usage_service;
CREATE DATABASE enrollment_service;
CREATE DATABASE product_service;
```

---

## Running Services in Dev Mode

Quarkus provides excellent developer experience with live reload:

```bash
cd customer-service
./mvnw quarkus:dev
```

Features available in dev mode:

- **Live reload** - Code changes apply automatically
- **Dev UI** - Available at `http://localhost:8080/q/dev`
- **Swagger UI** - Available at `http://localhost:8080/q/swagger-ui`

---

## Environment Variables

Common environment variables for development:

```bash
# Database
QUARKUS_DATASOURCE_JDBC_URL=jdbc:postgresql://localhost:5432/customer_service
QUARKUS_DATASOURCE_USERNAME=ossrep
QUARKUS_DATASOURCE_PASSWORD=ossrep

# Logging
QUARKUS_LOG_LEVEL=DEBUG
```
