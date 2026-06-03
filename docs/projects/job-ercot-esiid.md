# job-ercot-esiid

Quarkus CLI application that ingests ERCOT TDSP ESIID Extract data and pushes it to [api-service-point](api-service-point.md).

Uses [integration-ercot](integration-ercot.md) for ERCOT API access and CSV parsing.

## Technology Stack

| Technology        | Purpose                              |
| ----------------- | ------------------------------------ |
| Quarkus 3.33      | Application framework                |
| Java 25           | Language (latest LTS)                |
| Picocli           | CLI command framework                |
| PostgreSQL        | Ingestion log tracking               |
| Flyway            | Schema migration                     |
| REST Client       | Communication with api-service-point |
| integration-ercot | ERCOT data access and parsing        |

## How It Works

1. Lists available documents for ERCOT report 203 (TDSP ESI ID Extract) via public MIS endpoint
2. Checks local ingestion log database to skip already-processed files
3. Downloads new ZIP archives, parses the CSV contents
4. Sends parsed records in batches to the `api-service-point` bulk upsert endpoint
5. Records ingestion status (COMPLETED/FAILED) in the local database

## Data Flow

```
ERCOT MIS (public)
    │
    ▼
integration-ercot (list docs, download, parse ZIP/CSV)
    │
    ▼
job-ercot-esiid (orchestrate ingestion)
    │
    ├──▶ api-service-point  PUT /api/v1/service-points/bulk
    ├──▶ api-service-point  GET /api/v1/tdsps?code={code}
    │
    └──▶ local PostgreSQL (ingestion_log tracking)
```

## Database

The CLI maintains its own `ingestion_log` table to track which files have been processed.

| Column           | Type      | Description                              |
| ---------------- | --------- | ---------------------------------------- |
| ingestion_log_id | BIGSERIAL | Primary key                              |
| tdsp_id          | BIGINT    | TDSP identifier (from api-service-point) |
| file_name        | TEXT      | Archive file name (unique)               |
| file_type        | TEXT      | FUL (monthly full) or DAILY (daily delta)|
| record_count     | BIGINT    | Number of records processed              |
| status           | TEXT      | PROCESSING, COMPLETED, or FAILED         |
| started_at       | TIMESTAMP | When processing began                    |
| completed_at     | TIMESTAMP | When processing finished                 |
| error_message    | TEXT      | Error details on failure                 |

## Configuration

| Environment Variable       | Description                         |
| -------------------------- | ----------------------------------- |
| `API_SERVICE_POINT_URL`    | Base URL for api-service-point      |
| `DATASOURCE_JDBC_URL`      | PostgreSQL JDBC URL (prod)          |
| `DATASOURCE_USERNAME`      | Database username (prod)            |
| `DATASOURCE_PASSWORD`      | Database password (prod)            |

## Local Development

### Prerequisites

- Java 25
- Maven 3.9+
- Podman (for Dev Services on port 5431)
- A running `api-service-point` instance
- `integration-ercot` installed to local Maven repo

### Run

```shell
API_SERVICE_POINT_URL=http://localhost:8080 \
JAVA_HOME=/usr/lib/jvm/java-25-openjdk ./mvnw quarkus:run
```

### Build

```shell
JAVA_HOME=/usr/lib/jvm/java-25-openjdk ./mvnw package
```

### Container build

```shell
JAVA_HOME=/usr/lib/jvm/java-25-openjdk ./mvnw package
podman build -f Containerfile -t job-ercot-esiid .
```
