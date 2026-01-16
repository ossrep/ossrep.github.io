# OSSREP - Open Source Retail Energy Platform

## Overview

OSSREP is an open source microservices platform for retail electricity operations. It provides a comprehensive set of services for managing customers, meters, billing, market operations, and product offerings in a deregulated electricity market (such as ERCOT in Texas).

## Technology Stack

| Layer | Technology |
|-------|------------|
| API Gateway | Red Hat Connectivity Link |
| Service Mesh | Red Hat Service Mesh (Istio) |
| Backend Services | Java (Quarkus) |
| Frontend | Angular (standalone components, signals) |
| Database | PostgreSQL (one per service) |
| Messaging | Apache Kafka |
| Platform | OpenShift (Kubernetes) |
| Identity | Keycloak (OIDC/JWT) |
| Infrastructure | Ansible |

## Domain Model

### Entity Hierarchy

```
Customer
  └── Accounts (1+ for business, typically 1 for individual)
        └── Premises (1+)
              └── Meters (1+)

Customer
  └── Contracts (1+ over time)
        └── Plan (reference)
        └── Accounts (1+ covered by contract)
```

### Core Entities

#### Customer
- `customerId` (Long) - Unique identifier
- `type` (Enum) - `individual` or `business`
- `firstName`, `lastName` (String) - For individual customers
- `businessName` (String) - For business customers
- `email`, `phone` (String) - Contact information
- `status` (Enum) - `active`, `inactive`, `pending`
- `createdAt` (Timestamp)

#### Account
A billing entity that groups premises. All billing occurs at the account level.
- `accountId` (Long)
- `accountNumber` (String) - Human-readable
- `customerId` (Long) - Parent customer
- `nickname` (String) - Optional friendly name (e.g., "Headquarters")
- `status` (Enum) - `active`, `inactive`, `pending`
- `currentBalance` (Decimal)
- `createdAt` (Timestamp)

#### Premise
A physical location where utility services are provided.
- `premiseId` (Long)
- `accountId` (Long) - Parent account
- `street`, `unit`, `city`, `state`, `zip` (String) - Address
- `isPrimary` (Boolean)
- `serviceStartDate` (Date)

#### Meter
A physical device measuring electricity consumption.
- `meterId` (Long)
- `meterNumber` (String) - Physical identifier (e.g., "E-12345678")
- `premiseId` (Long) - Parent premise
- `type` (Enum) - `electric`
- `status` (Enum) - `available`, `active`, `inactive`
- `installDate` (Date)
- `lastReading` (Decimal)
- `lastReadingDate` (Date)
- `unit` (String) - `kWh`

#### Plan
A rate plan/product offering that determines pricing and contract terms.
- `planId` (Long)
- `name` (String) - Display name (e.g., "Green Energy 12")
- `description` (String) - Marketing description
- `ratePerKwh` (Decimal) - Energy rate in cents
- `termMonths` (Integer) - Contract length (1-24 months typical)
- `earlyTerminationFee` (Decimal) - Fee for early cancellation
- `baseCharge` (Decimal) - Monthly base/service charge
- `renewablePercent` (Integer) - 0-100, percentage renewable energy
- `planType` (Enum) - `fixed`, `variable`, `indexed`
- `features` (List<String>) - Marketing features
- `status` (Enum) - `active`, `inactive`, `archived`

Each plan requires regulatory disclosure documents:
- **EFL** (Electricity Facts Label) - Standardized pricing disclosure
- **TOS** (Terms of Service) - Contract terms and conditions
- **YRAC** (Your Rights as a Customer) - Consumer rights information

#### Contract
The formal agreement between customer and REP for electricity service.
- `contractId` (Long)
- `contractNumber` (String) - Human-readable (e.g., "CTR-2025-00001")
- `customerId` (Long)
- `planId` (Long) - The selected plan
- `startDate` (Date) - Service start date
- `endDate` (Date) - Calculated from startDate + plan.termMonths
- `status` (Enum) - `pending`, `active`, `expired`, `cancelled`, `renewed`
- `signedAt` (Timestamp) - When customer accepted terms
- `earlyTerminationFee` (Decimal) - From plan at time of signing
- `createdAt` (Timestamp)

A contract covers one or more accounts (many-to-many relationship).

#### Bill
An invoice generated for an account.
- `billId` (Long)
- `accountId` (Long)
- `amount` (Decimal)
- `dueDate` (Date)
- `status` (Enum) - `pending`, `unpaid`, `paid`, `overdue`
- `billingPeriodStart`, `billingPeriodEnd` (Date)
- `paidDate` (Date)
- `paidAmount` (Decimal)

## Customer Types

### Individual (Residential)
- Typically one account, one premise, one meter
- May have multiple premises (primary residence + vacation home)
- Simpler account structure

### Business (Commercial)
- Multiple accounts (by department, cost center, location group)
- Multiple premises per account
- Multiple meters per premise
- Aggregated billing and reporting across accounts
- Primary contact person required

## Microservices Architecture

### Customer Service
Central service for customer-related operations.
- Customer enrollment and management
- Account lifecycle operations
- Premise management
- Meter registration and tracking
- Contract management (creation, renewal, cancellation)
- Customer preferences and communication settings

### Meter Service
- Meter registration and tracking
- Meter reading collection
- Usage data validation (VEE)
- Interval data management

### Billing Service
- Invoice generation
- Payment processing
- Payment arrangements
- Collections management

### Market Service
- EDI transaction processing
- Market participant messaging (ERCOT)
- Enrollment transactions
- Regulatory reporting

### Product Service
- Rate plan definitions
- Product catalog with pricing
- Contract terms and ETF configuration
- Renewable energy options
- Regulatory document management (EFL, TOS, YRAC)
- Service area/ZIP code availability

## Customer Portal (Angular)

### Enrollment Flow (Start Service)

7-step guided enrollment with confirmation page:

**Step 1: Customer Type**
- Radio selection: Individual/Residential or Business/Commercial
- Card-based UI with icons

**Step 2: Contact Information**
- Individual: First name, last name, email, phone
- Business: Business name, type (dropdown), Tax ID/EIN, email, phone, plus primary contact person details

**Step 3: Service Address & Start Date**
- Street address, unit (optional), city, state (dropdown), ZIP
- Requested service start date (date picker, minimum tomorrow)
- Info note about 1-3 business day activation

**Step 4: Plan Selection**
- Loading state while fetching plans for ZIP code
- Plan cards showing:
  - Plan name with renewable badge (100% Renewable, X% Renewable)
  - Description
  - Feature badges
  - Rate per kWh (large, primary color)
  - Term length and base charge
  - Links to EFL, TOS, YRAC documents
  - Early termination fee (if applicable)
- Radio selection, card highlights when selected

**Step 5: Meter Selection**
- Loading state while fetching available meters at address
- List of meters with checkboxes
- Shows meter number, type badge (Electric), status badge (Available)
- Individual customers typically see 1 meter
- Business customers may see multiple meters
- Must select at least one meter to proceed
- Selected count displayed

**Step 6: Mailing Address**
- Checkbox: "Same as service address"
- If unchecked, show address form
- If checked, display service address as confirmation

**Step 7: Review & Terms**
- Bill delivery preference (Email/Paperless, Mail, Both)
- Marketing opt-in checkbox
- Terms of service scrollable box
- Acceptance checkbox (required)
- Enrollment summary card showing:
  - Customer name and type
  - Contact info
  - Service address and selected meters
  - Selected plan with rate, term, renewable badges
  - Contract details: start date, end date, term length
  - Early termination fee notice

**Confirmation Page (`/signup/confirmation`)**
- Success header with checkmark icon
- Unique confirmation number (e.g., "ENR-M5X2Y3-AB4C")
- Summary cards:
  - Customer Information
  - Service Address with enrolled meters
  - Plan details (rate, term, base charge, ETF)
  - Contract dates and bill delivery preference
- "What Happens Next" section with 3-step timeline:
  1. Confirmation email
  2. Service activation on start date
  3. First bill after billing cycle
- Print Summary button
- Sign In to Your Account link

### Progress Indicator
- Horizontal step indicators (circles with numbers/checkmarks)
- Progress bar showing completion percentage
- Step labels visible on larger screens

### UI/UX Details
- Bootstrap 5 styling
- Card-based selection with border highlight
- Left border accent on selected items (blue)
- Light blue background on selected items
- Smooth transitions
- Form validation with required field indicators
- Loading spinners for async operations
- Responsive design

### Dashboard (Authenticated)
- Aggregated view across all accounts
- Total balance due with next due date
- Account, premise, meter counts
- Usage summary by meter type

### Account Management
- View all accounts with balances
- Premises with meter details
- Meter reading history

### Profile & Preferences
- Update contact information
- Manage mailing address
- Communication preferences
- Notification settings

## API Endpoints

### Enrollment
- `POST /api/enrollment` - Submit enrollment
- `GET /api/enrollment/plans?zip={zipCode}` - Available plans
- `GET /api/enrollment/plans/{planId}` - Plan details with documents
- `GET /api/enrollment/meters?street={}&city={}&state={}&zip={}` - Available meters
- `POST /api/enrollment/validate-address` - Validate address

### Contracts
- `GET /api/contracts` - List customer contracts
- `GET /api/contracts/{contractId}` - Contract details
- `GET /api/contracts/active` - Active contract(s)
- `POST /api/contracts/{contractId}/renew` - Renew contract
- `POST /api/contracts/{contractId}/cancel` - Cancel (ETF may apply)

### Customer
- `GET /api/customer` - Current customer profile
- `PUT /api/customer` - Update profile
- `GET /api/customer/summary` - Aggregated summary

### Accounts
- `GET /api/accounts` - List accounts
- `GET /api/accounts/{accountId}` - Account details
- `PUT /api/accounts/{accountId}` - Update account
- `GET /api/accounts/{accountId}/premises` - Premises for account

### Meters
- `GET /api/premises/{premiseId}/meters` - Meters at premise
- `GET /api/meters/{meterId}` - Meter details
- `GET /api/meters/{meterId}/readings` - Reading history

### Preferences
- `GET /api/preferences` - Customer preferences
- `PUT /api/preferences` - Update preferences

## Events (Kafka)

| Event | Description |
|-------|-------------|
| `EnrollmentSubmitted` | New enrollment received |
| `EnrollmentCompleted` | Enrollment processed |
| `CustomerCreated` | New customer enrolled |
| `ContractCreated` | New contract created |
| `ContractActivated` | Contract start date reached |
| `ContractExpiring` | Contract approaching end |
| `ContractExpired` | Contract term ended |
| `ContractRenewed` | Customer renewed |
| `ContractCancelled` | Contract cancelled early |
| `MeterEnrolled` | Meter selected during enrollment |
| `PreferencesUpdated` | Preferences changed |

## Security

- OIDC/JWT authentication via Keycloak
- Customer ID derived from JWT `sub` claim
- Customers can only access their own data
- Account ownership verified on all operations
- Admin endpoints require elevated roles
- PII encrypted at rest
- Audit logging for data access

## Mock Data Examples

### Plans
1. **Green Energy 12** - 11.9¢/kWh, 12-month, $9.95 base, 100% wind, $150 ETF
2. **Value Saver 24** - 10.5¢/kWh, 24-month, $4.95 base, 15% renewable, $200 ETF
3. **Flex Power** - 13.2¢/kWh, month-to-month, no base, no ETF, variable rate
4. **Solar Plus 18** - 12.4¢/kWh, 18-month, $9.95 base, 100% solar, $175 ETF

### Meters
- Individual: Single meter (E-12345678)
- Business: Multiple meters (E-12345678, E-23456789, E-34567890)

### Confirmation Number Format
`ENR-{timestamp_base36}-{random_4chars}` (e.g., "ENR-M5X2Y3-AB4C")
