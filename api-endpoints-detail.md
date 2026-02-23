# RCM API Endpoints - Complete Reference
**Source:** `rcm-serverless/swagger.yaml` (OpenAPI 3.0)
**Generated:** 2026-02-22

## Summary
- **Total Endpoints:** 48 (from swagger)
- **Total Routes in API Gateway:** 84 (includes OPTIONS for CORS)
- **Domains:** core, creditblock, cybersweep, vendor
- **Auth:** Keycloak Token Authorizer

---

## Core Domain (`/core/v1`)

### Account & Device Management
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/core/v1/account/device/{userId}` | Get client's or client manager's device(s) |
| POST | `/core/v1/account/device` | Enroll device for push notifications |
| PATCH | `/core/v1/account/device/{deviceId}` | Update device settings |

### Client Management
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/core/v1/client/{clientId}` | Get client details |
| GET | `/core/v1/client/{clientId}/profile` | Get client profile |
| PATCH | `/core/v1/client/{clientId}/profile` | Update client profile |
| GET | `/core/v1/client/{clientId}/family` | List family members |
| GET | `/core/v1/client/{clientId}/family/{familyId}` | Get specific family member |
| PATCH | `/core/v1/client/{clientId}/family/{familyId}` | Update family member |
| POST | `/core/v1/client/{clientId}/support` | Create support case |

### Partner Management
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/core/v1/partner` | List all partners |
| GET | `/core/v1/partner/{partnerId}` | Get partner details |
| PATCH | `/core/v1/partner/{partnerId}` | Update partner |
| GET | `/core/v1/partner/{partnerId}/pricebook` | Get partner pricebook |
| GET | `/core/v1/partner/{partnerId}/registration` | Get registration settings |
| GET | `/core/v1/partner/contacts` | List partner contacts |
| POST | `/core/v1/partner/contacts` | Create partner contact |
| GET | `/core/v1/partner/contacts/{contactId}` | Get contact details |
| PATCH | `/core/v1/partner/contacts/{contactId}` | Update contact |
| GET | `/core/v1/partner/contacts/{contactId}/clients` | Get contact's clients |
| GET | `/core/v1/partner/client-managers/{contactId}/clients` | Get CM's clients |

### Array Integration (Credit Reports)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/core/v1/array/enrollment/{clientId}` | Get enrollment status |
| POST | `/core/v1/array/enrollment/report` | Generate enrollment report |
| POST | `/core/v1/array/monitoring/enroll` | Enroll in credit monitoring |
| POST | `/core/v1/array/pullcreditreport` | Pull credit report |
| POST | `/core/v1/array/register` | Register with Array |
| GET | `/core/v1/array/register/{arrayClientKey}` | Get registration details |
| POST | `/core/v1/array/listener` | Array webhook listener |

### Notifications
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/core/v1/notifications/client/{accountId}` | List notifications |
| GET | `/core/v1/notifications/client/{accountId}/{notificationId}` | Get notification |
| PATCH | `/core/v1/notifications/client/{accountId}/{notificationId}` | Update notification |

### Verified (Vouched KYC)
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/core/v1/verified` | Start verification |
| POST | `/core/v1/verified/case` | Create verification case |
| GET | `/core/v1/verified/case/{contactId}` | Get verification status |
| GET | `/core/v1/verified/businessManager/{contactId}` | Check BM verification |

### Controls
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/core/v1/controls/list/{object}/{field}` | Get picklist values from SF |

---

## CreditBlock Domain (`/creditblock/v1`)

### Bureau Freeze/Unfreeze
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/creditblock/v1/bureaus/summary/{clientId}` | Get bureau freeze summary |
| GET | `/creditblock/v1/bureaus/{bureau}/checkStatus` | Check freeze status |
| POST | `/creditblock/v1/bureaus/{bureau}/checkStatus` | Initiate status check |
| POST | `/creditblock/v1/bureaus/{bureau}/block` | Freeze bureau |
| POST | `/creditblock/v1/bureaus/{bureau}/unblock` | Unfreeze bureau |
| POST | `/creditblock/v1/primary/blockall` | Freeze all bureaus |
| POST | `/creditblock/v1/primary/unblockall` | Unfreeze all bureaus |
| GET | `/creditblock/v1/checkStatusResult/{token}` | Get async status result |

**Bureaus:** `experian`, `equifax`, `transunion`

---

## CyberSweep Domain (`/cybersweep/v1`)

### IDP (Identity Protection) Monitors
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/cybersweep/v1/idp/monitors/{accountId}` | List monitors for account |
| POST | `/cybersweep/v1/idp/monitors/{accountId}` | Create new monitor |
| GET | `/cybersweep/v1/idp/monitors/{accountId}/{monitorId}` | Get monitor details |
| PATCH | `/cybersweep/v1/idp/monitors/{accountId}/{monitorId}` | Update monitor |
| DELETE | `/cybersweep/v1/idp/monitors/{accountId}/{monitorId}` | Delete monitor |

### Website Automation
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/cybersweep/v1/website/queue_automation` | Queue website automation task |

---

## Vendor Domain (`/vendor`)

### Array v1 (Legacy)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/vendor/array/v1/report/credit-score-history` | Get score history |
| POST | `/vendor/array/v1/monitoring/enrollments` | Create monitoring enrollment |

### Array v2 (Current)
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/vendor/array/v2/user` | Create Array user |
| GET | `/vendor/array/v2/user/{userId}/monitoring-enrollments` | Get user's enrollments |
| PATCH | `/vendor/array/v2/authenticate` | Authenticate user |
| POST | `/vendor/array/v2/report/order` | Order credit report |
| GET | `/vendor/array/v2/report/pdf` | Get PDF report |
| PUT | `/vendor/array/v2/report/token` | Refresh report token |

---

## Lambda Architecture

### Submodule → Lambda Mapping

```
packages/
├── core-submodule/
│   ├── AccountLambda              → core-cdk-lambda-account-*
│   ├── ContactLambda              → core-cdk-lambda-contact-*
│   ├── ClientFamilyLambda         → core-cdk-lambda-client-family-*
│   ├── KeycloakAuthorizerLambda   → core-cdk-lambda-keycloak-authorizer-*
│   ├── ArrayWebhook*Lambda        → core-cdk-lambda-array-webhook-*
│   ├── PersonNotificationsLambda  → core-cdk-lambda-notifications-*
│   ├── VouchedVerifiedLambda      → core-cdk-lambda-vouched-*
│   └── ...
│
├── cb-submodule/
│   ├── BureauCaseLambda           → creditblock-cdk-lambda-bureauCase-*
│   ├── BureauBlockLambda          → creditblock-cdk-lambda-bureau-block-*
│   ├── BureauUnblockLambda        → creditblock-cdk-lambda-bureau-unblock-*
│   └── ...
│
├── cs-submodule/
│   ├── IDPMonitorLambda           → cybersweep-cdk-lambda-idp-*
│   ├── WebsiteAutomationLambda    → cybersweep-cdk-lambda-website-*
│   └── ...
│
└── vendor-submodule/
    ├── ArrayUserLambda            → vendor-cdk-lambda-array-user-*
    ├── ArrayReportLambda          → vendor-cdk-lambda-array-report-*
    └── ...
```

### Layers Used
- `SalesforceLayer` — SF API client, SOQL helpers
- `PuppeteerChromiumLayer` — Browser automation (CyberSweep)
- `ParameterStoreLayer` — AWS Parameter Store extension

### Step Functions (State Machines)
- `ArrayWebhookStateMachine` — Process Array webhook events
- `ArrayPullCreditReportSchedulerStateMachine` — Scheduled report pulls
- `ArrayFailedOneBReportSchedulerStateMachine` — Retry failed reports
- `ArrayClientCreditReportStateMachine` — Credit report workflow
- `ArrayClientRegistrationStateMachine` — Client registration flow
- `ArrayGetEnrollmentStateMachine` — Get enrollment workflow

---

## Data Schemas (from swagger)

### Core Entities
- **Client**: id, firstName, lastName, email, phone, accountId, ...
- **ClientsFamily**: id, firstName, lastName, relationship, ...
- **ClientProfile**: extended client data
- **Client_Device**: id, userId, token, deviceType, platform, ...

### Partner Entities
- **Partner**: id, name, type, status, ...
- **PartnerContact**: id, firstName, lastName, email, role, ...
- **PartnerPricebook**: pricing configuration
- **PartnerRegistration**: registration settings

### Service Entities
- **Notification**: id, type, message, read, createdAt, ...
- **BureauCase**: id, clientId, bureau, status, ...
- **Idp_Monitoring**: id, accountId, monitorType, status, ...

### Vendor Entities
- **ArrayClientData**: Array-specific client data
- **ArrayGetClientEnrollment**: enrollment status

---

## Authentication

All endpoints (except webhooks) require Keycloak authentication:

```yaml
securitySchemes:
  KeycloakAuthorizer:
    type: apiKey
    name: authorizationToken
    in: header
    x-amazon-apigateway-authtype: custom
```

**Token flow:**
1. Client authenticates with Keycloak SSO
2. Gets JWT access token
3. Passes token in `authorizationToken` header
4. `KeycloakAuthorizerLambda` validates token
5. Returns IAM policy with scopes

---

## Environment URLs

| Environment | API Base URL | Keycloak |
|-------------|--------------|----------|
| Development | https://api.dev.regalcredit.com | sso.dev.regalcredit.com |
| Staging | https://api.stg.regalcredit.com | sso.stg.regalcredit.com |
| Production | https://api.regalcredit.com | sso.regalcredit.com |

---

*Generated from rcm-serverless repo + AWS API Gateway scan*
