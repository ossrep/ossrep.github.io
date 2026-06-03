# api-service-point

Utility/energy service point management REST API built with Quarkus.

## Technology Stack

| Technology   | Purpose                   |
| ------------ | ------------------------- |
| Quarkus 3.33 | Application framework     |
| Java 25      | Language (latest LTS)     |
| PostgreSQL   | Persistence               |
| Kafka        | Event streaming           |
| Flyway       | Schema migration          |
| OIDC         | Authentication            |

## Architecture

The API follows a three-tier architecture:

- **API layer** (`com.ossrep.servicepoint.api`) - REST resources, request/response records, roles
- **Service layer** (`com.ossrep.servicepoint.service`) - Business logic, domain records, event emission
- **Repository layer** (`com.ossrep.servicepoint.repository`) - JPA entities, Panache repositories

## REST Endpoints

All endpoints are secured with OIDC and require appropriate roles.

### ISOs - `/api/v1/isos`

| Method   | Path          | Roles        | Description         | Status |
| -------- | ------------- | ------------ | ------------------- | ------ |
| `GET`    | `/`           | admin, user  | List all ISOs       | 200    |
| `GET`    | `/{isoId}`    | admin, user  | Get an ISO by ID    | 200    |

### TDSPs - `/api/v1/tdsps`

| Method   | Path           | Roles        | Description                        | Status |
| -------- | -------------- | ------------ | ---------------------------------- | ------ |
| `GET`    | `/`            | admin, user  | List all TDSPs (filter: `?code=`)  | 200    |
| `GET`    | `/{tdspId}`    | admin, user  | Get a TDSP by ID                   | 200    |

### Service Points - `/api/v1/service-points`

| Method   | Path                   | Roles        | Description                  | Status |
| -------- | ---------------------- | ------------ | ---------------------------- | ------ |
| `GET`    | `/`                    | admin, user  | List all service points      | 200    |
| `GET`    | `/{servicePointId}`    | admin, user  | Get a service point by ID    | 200    |
| `POST`   | `/`                    | admin        | Create a new service point   | 201    |
| `PUT`    | `/{servicePointId}`    | admin        | Update a service point       | 200    |
| `PUT`    | `/bulk`                | admin        | Bulk upsert by ESIID         | 200    |
| `DELETE` | `/{servicePointId}`    | admin        | Delete a service point       | 204    |

## Domain Model

### ISO

| Field       | Type    | Required | Description                                    |
| ----------- | ------- | -------- | ---------------------------------------------- |
| isoId       | Long    | auto     | Primary key (BIGSERIAL)                        |
| name        | String  | yes      | Full name of the ISO                           |
| code        | String  | yes      | Short code (unique, e.g. ERCOT)                |
| description | String  | no       | Description                                    |
| createdAt   | Instant | auto     | Creation timestamp                             |
| updatedAt   | Instant | auto     | Last update timestamp                          |

### TDSP

| Field       | Type    | Required | Description                                    |
| ----------- | ------- | -------- | ---------------------------------------------- |
| tdspId      | Long    | auto     | Primary key (BIGSERIAL)                        |
| isoId       | Long    | yes      | Foreign key to ISO                             |
| name        | String  | yes      | Full name of the TDSP                          |
| code        | String  | yes      | Short code (unique, e.g. CENTERPOINT)          |
| duns        | String  | no       | DUNS number (unique when present)              |
| description | String  | no       | Description                                    |
| createdAt   | Instant | auto     | Creation timestamp                             |
| updatedAt   | Instant | auto     | Last update timestamp                          |

### ServicePoint

The service point model is aligned with the official [ERCOT TDSP ESIID Extract DDL](https://www.ercot.com/mp/data-products/data-product-details?id=ZP15-612) (Report Type ID 203).

| Field                 | Type    | Required | Description                                                    |
| --------------------- | ------- | -------- | -------------------------------------------------------------- |
| servicePointId        | Long    | auto     | Primary key (BIGSERIAL)                                        |
| tdspId                | Long    | no       | Foreign key to TDSP                                            |
| esiid                 | String  | yes      | Electric Service Identifier (22-char ERCOT ID, unique)         |
| street                | String  | no       | Registered street address                                      |
| streetLine2           | String  | no       | Address overflow                                               |
| city                  | String  | no       | City                                                           |
| state                 | String  | no       | State                                                          |
| zip                   | String  | no       | ZIP code                                                       |
| county                | String  | no       | County                                                         |
| meterReadCycle        | String  | no       | Meter read cycle / structure type                              |
| status                | String  | yes      | Active, Inactive, or De-Energized                              |
| premiseType           | String  | no       | Residential, Small Non-Residential, or Large Non-Residential   |
| stationCode           | String  | no       | Substation code                                                |
| stationName           | String  | no       | Substation name                                                |
| metered               | Boolean | yes      | Whether the service point is metered                           |
| openServiceOrders     | String  | no       | Concatenated pending service orders (pipe-delimited)           |
| polrCustomerClass     | String  | no       | POLR customer class (Residential, Small/Medium/Large Non-Res)  |
| settlementAmsIndicator| String  | no       | AMS meter flag (Y/N)                                           |
| tdspAmsIndicator      | String  | no       | TDSP AMS indicator (AMSM, AMSR, or null)                       |
| switchHoldIndicator   | String  | no       | Switch hold present at ERCOT (Y/N)                             |
| meteredServiceType    | String  | no       | Primary metered service type code (e.g. M04)                   |
| meteredServiceTypeDesc| String  | no       | Description of the metered service type                        |
| createdAt             | Instant | auto     | Creation timestamp                                             |
| updatedAt             | Instant | auto     | Last update timestamp                                          |

## Kafka Events

Topic: `service-point-events`

Events are emitted on CRUD operations:

| Event Type              | Trigger              |
| ----------------------- | -------------------- |
| SERVICE_POINT_CREATED   | New service point    |
| SERVICE_POINT_UPDATED   | Service point update |
| SERVICE_POINT_DELETED   | Service point delete |

Each event contains the event type, a timestamp, and the full service point payload.

## Database

### Tables

- `iso` - Independent System Operator reference data (seeded with ERCOT)
- `tdsp` - Transmission/Distribution Service Provider reference data (seeded with 11 TDSPs)
- `service_point` - Service points with `tdsp_id` FK to `tdsp`


### Key Conventions

- Singular table names
- Primary keys: `BIGSERIAL`
- Foreign keys: `BIGINT REFERENCES`
- `service_point_id` sequence starts at 1000000000000000000
- Schema managed by Flyway
- Test data loaded in dev/test profiles via `db/testdata`

## Local Development

### Prerequisites

- Java 25
- Maven 3.9+
- Podman (for Dev Services)

### Run in dev mode

```shell
cd api-service-point
JAVA_HOME=/usr/lib/jvm/java-25-openjdk ./mvnw quarkus:dev
```

Quarkus Dev Services will automatically start PostgreSQL (port 5431) and Kafka containers.

The API will be available at [http://localhost:8080](http://localhost:8080).

OpenAPI spec available at [http://localhost:8080/q/openapi](http://localhost:8080/q/openapi).

### Build

```shell
JAVA_HOME=/usr/lib/jvm/java-25-openjdk ./mvnw package
```

### Container build

```shell
JAVA_HOME=/usr/lib/jvm/java-25-openjdk ./mvnw package
podman build -f Containerfile -t api-service-point .
```
