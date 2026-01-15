# Customer Service

The Customer Service manages customer lifecycle, accounts, premises, and meters. It is the central service for customer-related operations in OSSREP.

## Overview

**Repository:** `customer-service`

**Key Responsibilities:**

- Customer enrollment and management
- Account lifecycle operations
- Premise management
- Meter registration and tracking
- Customer preferences and communication settings

## Domain Model

See [Domain Model](../architecture/domain-model.md) for detailed entity descriptions and relationships.

### Entity Hierarchy

```
Customer → Accounts → Premises → Meters
```

## Customer Types

The service supports two customer types with different organizational structures:

### Individual Customers

Residential consumers with simpler account structures:

- Typically one account
- One or more premises (primary residence, vacation homes, rentals)
- One or more meters per premise

### Business Customers

Commercial/industrial customers with complex organizational needs:

- Multiple accounts (by department, cost center, location group)
- Multiple premises per account
- Multiple meters per premise
- Aggregated billing and reporting across all accounts

## API Endpoints

### Customer Operations

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/customer` | Get current customer profile |
| `PUT` | `/api/customer` | Update customer profile |
| `GET` | `/api/customer/summary` | Get aggregated customer summary |

### Account Operations

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/accounts` | List all accounts for customer |
| `GET` | `/api/accounts/{accountId}` | Get account details |
| `PUT` | `/api/accounts/{accountId}` | Update account (nickname, etc.) |
| `GET` | `/api/accounts/{accountId}/summary` | Get account summary with balances |

### Premise Operations

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/accounts/{accountId}/premises` | List premises for account |
| `GET` | `/api/premises/{premiseId}` | Get premise details |
| `PUT` | `/api/premises/{premiseId}` | Update premise |
| `POST` | `/api/premises/{premiseId}/set-primary` | Set as primary premise |

### Meter Operations

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/premises/{premiseId}/meters` | List meters at premise |
| `GET` | `/api/meters/{meterId}` | Get meter details |
| `GET` | `/api/meters/{meterId}/readings` | Get meter reading history |

### Preferences Operations

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/preferences` | Get customer preferences |
| `PUT` | `/api/preferences` | Update preferences |
| `GET` | `/api/preferences/notifications` | Get notification settings |
| `PUT` | `/api/preferences/notifications` | Update notification settings |

## Data Models

### Customer Response

```json
{
  "customerId": 1,
  "type": "individual",
  "firstName": "John",
  "lastName": "Smith",
  "email": "john.smith@example.com",
  "phone": "(555) 123-4567",
  "status": "active",
  "createdAt": "2020-03-15T00:00:00Z",
  "accounts": [
    {
      "accountId": 1001,
      "accountNumber": "1234567890",
      "status": "active"
    }
  ]
}
```

### Business Customer Response

```json
{
  "customerId": 2,
  "type": "business",
  "businessName": "Acme Corporation",
  "email": "accounts@acme.com",
  "phone": "(555) 987-6543",
  "status": "active",
  "createdAt": "2019-01-15T00:00:00Z",
  "accounts": [
    {
      "accountId": 1002,
      "accountNumber": "9876543210",
      "nickname": "Headquarters",
      "status": "active"
    },
    {
      "accountId": 1003,
      "accountNumber": "5555555555",
      "nickname": "Warehouse District",
      "status": "active"
    }
  ]
}
```

### Account Detail Response

```json
{
  "accountId": 1001,
  "accountNumber": "1234567890",
  "nickname": null,
  "status": "active",
  "currentBalance": 142.50,
  "lastPaymentDate": "2024-12-15",
  "lastPaymentAmount": 128.75,
  "premises": [
    {
      "premiseId": 2001,
      "street": "123 Main Street",
      "unit": null,
      "city": "Anytown",
      "state": "TX",
      "zip": "75001",
      "isPrimary": true,
      "meters": [
        {
          "meterId": 3001,
          "meterNumber": "E-12345678",
          "type": "electric",
          "status": "active"
        }
      ]
    }
  ]
}
```

### Customer Summary Response

```json
{
  "totalAccounts": 3,
  "totalPremises": 5,
  "totalMeters": 6,
  "totalCurrentBalance": 5327.55,
  "totalDueAmount": 5327.55,
  "nextDueDate": "2025-01-22",
  "usageByType": {
    "electric": {
      "usage": 109280,
      "unit": "kWh"
    }
  }
}
```

### Meter Detail Response

```json
{
  "meterId": 3001,
  "meterNumber": "E-12345678",
  "type": "electric",
  "status": "active",
  "installDate": "2020-03-15",
  "lastReading": 1245,
  "lastReadingDate": "2025-01-10",
  "unit": "kWh",
  "premise": {
    "premiseId": 2001,
    "street": "123 Main Street",
    "city": "Anytown",
    "state": "TX",
    "zip": "75001"
  }
}
```

### Preferences Response

```json
{
  "billDelivery": "email",
  "notifications": {
    "billReady": true,
    "paymentDue": true,
    "outageAlerts": true,
    "highUsageAlerts": false
  },
  "autoPay": {
    "enabled": false,
    "paymentMethodId": null
  },
  "mailingAddress": {
    "sameAsPrimaryPremise": true,
    "street": null,
    "city": null,
    "state": null,
    "zip": null
  }
}
```

## Events

The Customer Service publishes events to Kafka for cross-service communication:

| Event | Description | Consumers |
|-------|-------------|-----------|
| `CustomerCreated` | New customer enrolled | Billing, Market |
| `CustomerUpdated` | Customer profile changed | Billing |
| `AccountCreated` | New account added | Billing |
| `AccountStatusChanged` | Account activated/deactivated | Billing, Market |
| `PremiseAdded` | New premise added | Meter, Market |
| `MeterInstalled` | Meter added to premise | Meter, Billing |
| `MeterRemoved` | Meter removed from premise | Meter, Billing |
| `PreferencesUpdated` | Customer preferences changed | Billing, Notifications |

## Integration Points

### Billing Service

- Receives account and customer data for invoice generation
- Notified of account status changes
- Queries customer preferences for bill delivery

### Meter Service

- Receives meter registration data
- Reports meter readings back to Customer Service
- Coordinates meter exchange operations

### Market Service

- Receives customer/premise data for market enrollments
- Coordinates move-in/move-out transactions
- Reports market-initiated changes

## Customer Portal Features

The Customer Portal (`ossrep-customer-portal`) provides self-service access to Customer Service capabilities:

### Dashboard

- Aggregated view across all accounts (especially for business customers)
- Total balance due with next due date
- Account/premise/meter counts
- Usage summary by meter type

### Account Management

- View all accounts with balances
- Premises with meter details
- Meter reading history

### Profile & Preferences

- Update contact information
- Manage mailing address
- Set communication preferences
- Configure notification settings
- Enable/disable auto-pay

## Security

### Authentication

- OIDC/JWT authentication via Keycloak
- Customer ID derived from JWT `sub` claim

### Authorization

- Customers can only access their own data
- Account ownership verified on all operations
- Admin endpoints require elevated roles

### Data Privacy

- PII encrypted at rest
- Audit logging for all data access
- GDPR/CCPA compliance considerations
