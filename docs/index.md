# Welcome to OSSREP

OSSREP organization documentation site.

## Projects

| Project                                                    | Description                                 | Tech Stack                        |
| ---------------------------------------------------------- | ------------------------------------------- | --------------------------------- |
| [api-service-point](projects/api-service-point.md)         | Utility service point management REST API   | Quarkus, PostgreSQL, Kafka        |
| [integration-ercot](projects/integration-ercot.md)         | ERCOT public data integration library       | Quarkus library                   |

## Local Development Ports

| Port  | Service                                | Project            |
| ----- | -------------------------------------- | ------------------ |
| 8001  | MkDocs dev server                      | ossrep.github.io   |
| 8080  | REST API                               | api-service-point  |
| 5431  | PostgreSQL (Dev Services)              | api-service-point  |

Port conventions:

- **8000s** -- web / documentation servers
- **8080s** -- API services
- **5430s** -- PostgreSQL databases (last digit matches the API it backs)
