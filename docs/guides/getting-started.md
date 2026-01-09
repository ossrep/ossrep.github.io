# Getting Started

Welcome to OSSREP! This guide will help you understand the project and get started with development.

## Project Structure

OSSREP is organized as a collection of repositories under the `ossrep` GitHub organization:

```
ossrep/
├── ossrep.github.io      # Documentation (this site)
├── customer-service      # Customer management
├── billing-service       # Billing and invoicing
├── usage-service         # Usage data processing
├── enrollment-service    # Enrollment and ERCOT
├── product-service       # Products and pricing
├── gateway-service       # API gateway
├── customer-portal       # Customer web application
├── admin-portal          # Operations web application
└── ossrep-commons        # Shared libraries
```

## Prerequisites

Before you begin, ensure you have the following installed:

| Tool | Version | Purpose |
|------|---------|---------|
| JDK | 21+ | Java development |
| Maven | 3.9+ | Build tool |
| Node.js | 20+ | Frontend development |
| Docker | 24+ | Containerization |
| Git | 2.40+ | Version control |

## Quick Start

### 1. Clone the repositories

```bash
# Create project directory
mkdir ossrep && cd ossrep

# Clone all repositories
git clone https://github.com/ossrep/ossrep.github.io.git
git clone https://github.com/ossrep/customer-service.git
# ... clone other repositories as needed
```

### 2. Start infrastructure

```bash
# Start PostgreSQL and other infrastructure
docker-compose up -d postgres
```

### 3. Run a service

```bash
cd customer-service
./mvnw quarkus:dev
```

The service will be available at `http://localhost:8080`.

### 4. Run the frontend

```bash
cd customer-portal
npm install
npm start
```

The application will be available at `http://localhost:4200`.

## Next Steps

- Read the [Development Setup](development-setup.md) guide for detailed environment configuration
- Review the [Architecture Overview](../architecture/overview.md) to understand the system design
- Check individual service READMEs for service-specific documentation
