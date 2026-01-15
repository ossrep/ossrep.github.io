# Open Source Retail Energy Platform

Welcome to **ossrep** - an open source microservices platform designed for retail electricity operations.

## Overview

OSSREP provides a comprehensive set of services for managing retail electricity operations including:

- **Customer Management** - Individual and business customer support, multi-account hierarchies, service address and meter management
- **Customer Enrollment** - 7-step guided enrollment flow with plan selection, meter selection, and confirmation page
- **Product/Plan Management** - Rate plans with fixed, variable, and indexed pricing; renewable energy options; regulatory documents (EFL, TOS, YRAC)
- **Metering** - Electric meter data collection, validation, and management
- **Billing** - Invoice generation, payment processing, and collections
- **Market Operations** - EDI transactions, market messaging, and regulatory compliance

## Goals

- Provide an open source reference implementation for retail electricity software
- Demonstrate modern microservices architecture patterns
- Enable rapid development and deployment of retail electricity solutions
- Support multiple market jurisdictions and regulatory requirements

## Technology Stack

| Layer          | Technology                    |
| -------------- | ----------------------------- |
| API Gateway    | Red Hat Connectivity Link     |
| Service Mesh   | Red Hat Service Mesh          |
| Backend        | Java (Quarkus)                |
| Frontend       | Angular                       |
| Database       | PostgreSQL                    |
| Messaging      | Apache Kafka                  |
| Platform       | OpenShift                     |
| Identity       | Keycloak (OIDC)               |
| Infrastructure | Ansible                       |
| Cloud          | Red Hat OpenShift on AWS      |

## Getting Started

See the [Development Setup](getting-started/development-setup.md) guide to begin contributing.
