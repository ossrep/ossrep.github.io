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

Base path: `/api/service-points`

| Method   | Path                   | Roles        | Description                  | Status |
| -------- | ---------------------- | ------------ | ---------------------------- | ------ |
| `GET`    | `/`                    | admin, user  | List all service points      | 200    |
| `GET`    | `/{servicePointId}`    | admin, user  | Get a service point by ID    | 200    |
| `POST`   | `/`                    | admin        | Create a new service point   | 201    |
| `PUT`    | `/{servicePointId}`    | admin        | Update a service point       | 200    |
| `DELETE` | `/{servicePointId}`    | admin        | Delete a service point       | 204    |

## Domain Model

The service point model is based on the ERCOT (Electric Reliability Council of Texas) TDSP service point structure, aligned with CenterPoint Energy reporting.

### ServicePoint

| Field              | Type    | Required | Description                                           |
| ------------------ | ------- | -------- | ----------------------------------------------------- |
| servicePointId     | Long    | auto     | Primary key (BIGSERIAL)                               |
| esiid              | String  | yes      | Electric Service Identifier (22-char ERCOT ID, unique)|
| street             | String  | no       | Street address                                        |
| streetLine2        | String  | no       | Address line 2                                        |
| city               | String  | no       | City                                                  |
| state              | String  | no       | State                                                 |
| zip                | String  | no       | ZIP code                                              |
| county             | String  | no       | County                                                |
| tdspDuns           | String  | no       | TDSP DUNS number (e.g. 957877905 for CenterPoint)     |
| townCode           | String  | no       | Town/district code                                    |
| status             | String  | yes      | Active, Inactive, or De-Energized                     |
| premiseType        | String  | no       | Residential or Small Non-Residential                  |
| powerRegion        | String  | no       | Power region (e.g. ERCOT)                             |
| stationCode        | String  | no       | Substation code                                       |
| stationName        | String  | no       | Substation name                                       |
| metered            | Boolean | yes      | Whether the service point is metered                  |
| pendingTransaction | String  | no       | Pending transaction (date and type)                   |
| polr               | Boolean | yes      | Provider of Last Resort flag                          |
| meterType          | String  | no       | Meter type (AMSM, AMSR)                               |
| createdAt          | Instant | auto     | Creation timestamp                                    |
| updatedAt          | Instant | auto     | Last update timestamp                                 |

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

- Table: `service_point` (singular naming)
- Primary key: `service_point_id` (BIGSERIAL, sequence starts at 1000000000000000000)
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

Quarkus Dev Services will automatically start PostgreSQL (port 5430) and Kafka containers.

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
