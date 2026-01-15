# Identity & Access Management

OSSREP uses [Red Hat build of Keycloak](https://access.redhat.com/products/red-hat-build-of-keycloak) in a dual-instance architecture to separate external and internal user populations, aligning with the [dual-gateway pattern](overview.md#gateway-architecture).

## Architecture Overview

```
┌─────────────────────┐                    ┌─────────────────────┐
│   Customer Portal   │                    │    Admin Portal     │
│     Mobile Apps     │                    │                     │
└──────────┬──────────┘                    └──────────┬──────────┘
           │                                          │
           ▼                                          ▼
┌─────────────────────┐                    ┌─────────────────────┐
│  Keycloak External  │                    │  Keycloak Internal  │
│                     │                    │                     │
│  Realm: ossrep      │                    │  Realm: ossrep-     │
│                     │                    │         internal    │
│  Users:             │                    │                     │
│  - Customers        │                    │  Users:             │
│  - Partners         │                    │  - Employees        │
│  - Third-party apps │                    │  - Administrators   │
└─────────────────────┘                    └─────────────────────┘
```

## Why Two Keycloak Instances?

| Concern | Benefit of Separation |
|---------|----------------------|
| **Security isolation** | Compromise of external IdP doesn't affect internal systems |
| **Policy flexibility** | Different password policies, MFA requirements, session timeouts |
| **User experience** | Tailored login flows for each audience |
| **Compliance** | Easier audit trails and access reviews per user population |
| **Availability** | External traffic spikes don't impact internal operations |

## Instance Configuration

### External Keycloak

Serves customer-facing applications and public APIs.

| Setting | Value |
|---------|-------|
| Port | 8180 (dev) |
| Realm | `ossrep` |
| Admin Console | http://localhost:8180 |

**Clients:**

- `ossrep-customer-portal` - Customer self-service web application
- `ossrep-mobile` - iOS and Android mobile apps
- `ossrep-api` - Third-party API consumers

### Internal Keycloak

Serves administrative applications and internal APIs.

| Setting | Value |
|---------|-------|
| Port | 8181 (dev) |
| Realm | `ossrep-internal` |
| Admin Console | http://localhost:8181 |

**Clients:**

- `ossrep-admin-portal` - Internal operations and customer service
- `ossrep-service-accounts` - Service-to-service authentication

## Local Development Setup

### Starting Both Keycloak Instances

The easiest way to start both Keycloak instances (along with PostgreSQL and Kafka) is using podman-compose from the `ossrep.github.io` repository:

```bash
# Authenticate with Red Hat registry (requires free developer account)
podman login registry.redhat.io

cd ossrep.github.io
podman-compose up -d
```

Both Keycloak instances automatically import pre-configured realms with test users on startup.

### Pre-configured Test Users

#### External Keycloak (http://localhost:8180)

Realm: `ossrep`

| Username | Password | Role | Description |
|----------|----------|------|-------------|
| alice | password | customer | Standard customer |
| bob | password | customer | Standard customer |
| carol | password | partner | Partner organization |

#### Internal Keycloak (http://localhost:8181)

Realm: `ossrep-internal`

| Username | Password | Role | Description |
|----------|----------|------|-------------|
| jsmith | password | csr | Customer service rep |
| mjones | password | supervisor | Team supervisor |
| admin | password | admin | System administrator |

### Admin Console Access

Both instances have admin consoles available:

- **External**: http://localhost:8180 (admin / admin)
- **Internal**: http://localhost:8181 (admin / admin)

### Realm Configuration Files

The realm configurations are stored in `keycloak/` and mounted into the containers:

- `keycloak/ossrep-realm.json` - External realm for customers/partners
- `keycloak/ossrep-internal-realm.json` - Internal realm for employees/admins

To modify the default configuration, edit these files and restart the containers:

```bash
podman-compose down
podman-compose up -d
```

!!! note "Realm Import Behavior"
    Keycloak only imports realms if they don't already exist. To re-import after changes, 
    remove the existing realm via the admin console or delete the container volumes:
    `podman-compose down -v`

## Roles and Permissions

### External Realm Roles

| Role | Description |
|------|-------------|
| `customer` | Standard customer access |
| `partner` | Partner organization access |
| `api-consumer` | Third-party API access |

### Internal Realm Roles

| Role | Description |
|------|-------------|
| `csr` | Customer service representative |
| `supervisor` | Team supervisor with escalation access |
| `admin` | System administrator |
| `readonly` | Read-only access for auditors |

## Service Integration

All backend services validate JWT tokens from both Keycloak instances. Services determine which issuer to trust based on the API gateway the request arrived through:

- Requests via **External Gateway** → validate against External Keycloak
- Requests via **Internal Gateway** → validate against Internal Keycloak

```yaml
# Example Quarkus configuration
quarkus:
  oidc:
    auth-server-url: ${OIDC_ISSUER:http://localhost:8180/realms/ossrep}
    client-id: ${OIDC_CLIENT_ID:ossrep-backend}
```

## Production Considerations

!!! warning "Development vs Production"
    The `start-dev` mode is for local development only. Production deployments should use:
    
    - Externalized PostgreSQL database
    - TLS/HTTPS configuration
    - Proper admin credentials
    - High availability configuration

For production deployment patterns, refer to the [Red Hat build of Keycloak documentation](https://access.redhat.com/documentation/en-us/red_hat_build_of_keycloak/).
