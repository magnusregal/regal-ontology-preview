# Regal Ontology v2

**Version:** 2.1  
**Last Updated:** 2026-02-23  
**Sources:** Regal Ontology v1, AWS Architecture Inventory v2/v2.1, ontology-map.aws.yaml, Product Hub codebase review

---

## 1) Purpose / Point

The Regal Ontology defines the canonical model of Regal's business domain (clients, partners, services, workflows, and systems) in a way that can be:

- **queried** (what is true right now?)
- **explained** (why did this happen?)
- **acted on** (what should we do next?)
- **automated safely** (what systems can mutate what records?)

It serves as the shared language across:

- Client Dashboard + Product Hub + B2C apps
- Core APIs (RCM Serverless)
- Salesforce (legacy / transitional source-of-truth)
- RDS/DynamoDB (target architecture)
- Vendors (Array, CyberSweep)
- Identity + permissions (Keycloak scopes)

---

## 2) Business Domains

Regal's services are organized into three primary business domains:

### 🏦 Credit Development
Building and improving credit profiles for clients.

| Entity | Description | System |
|--------|-------------|--------|
| Client | Person Account; primary customer entity | Gobodo, RDS, Salesforce |
| Lead | Prospective client; sales pipeline entry | Gobodo, RDS |
| Opportunity | Sales opportunity linked to Lead/Client | Gobodo, RDS |
| Account | Firm (Partner) or Individual; organizational entity | Gobodo, RDS |
| Contact | Person at a firm (PartnerContact) | Gobodo, RDS |
| Contract | Service agreement with Client/Partner | Gobodo |
| Product | Service offering (Credit Block, Monitoring, etc.) | Gobodo, Stripe |
| Pricebook | Partner-specific pricing; `accountId`, `partnerId` | Gobodo |
| ClientPriceOverride | Per-client pricing exception | Gobodo |
| Invoice | Billing record | Stripe |

### 🔧 Credit Repair
Active remediation of credit issues through bureau interactions.

| Entity | Description | System |
|--------|-------------|--------|
| BureauCase | Block/unblock request to credit bureaus | Gobodo, Array |
| CreditReport | Credit report from Array | Array API |
| ArrayUser | Client's Array vendor identity | Array |
| ArrayMonitoringEnrollment | Active monitoring subscription | Array |
| SupportCase | Customer support ticket | Gobodo, RDS |
| Task | Action item / follow-up | Gobodo, RDS |
| Event | Calendar/scheduled event | Gobodo, RDS |
| Note | Free-form annotation on any record | Gobodo |

### 🛡️ Identity Protection
Monitoring and protecting client identity across digital footprint.

| Entity | Description | System |
|--------|-------------|--------|
| CyberSweepClient | Client's CyberSweep enrollment record | CyberSweep |
| CyberSweepMonitor | Active identity monitor | CyberSweep API |
| CSAudit | CyberSweep audit/scan result | CyberSweep |
| PAWRecord | PII/audit record (Personal Audit Workflow) | CyberSweep |
| Verification | Identity verification (Vouched) | Vouched API |
| Notification | Alert to client/manager | Gobodo |
| Report | Analytics/compliance report | Gobodo |

---

## 3) Systems (Where Truth Lives / Where Actions Occur)

### Identity & Authorization

| System | AWS / Implementation | Notes |
|--------|----------------------|-------|
| Keycloak | keycloak-development (dev), keycloak-v16-release (prod) | Aurora MySQL; sso.stg.regalcredit.com, sso.release.regalcredit.com |
| KeycloakAuthorizer | core-cdk-lambda-keycloak-authorizer-* | Governs API access; scopes as capabilities (salesforce:read, client-dashboard:read-write) |

### Core Data / Business Systems

| System | AWS / Implementation | Notes |
|--------|----------------------|-------|
| Salesforce | External | Legacy primary for Person Accounts, Contacts, Partner Accounts, Cases, picklists |
| RDS (sfdatasync) | rcm-dsync-data-main-rds-dev/stg (865942198740 us-east-1) | Target SoR; schema: accounts, contacts, leads, cases, tasks, events, users, products, pricebooks |
| RDS Proxy | rcm-dsync-data-rds-proxy-stg | Used by rcm-dsync Lambdas |
| Gobodo | rcm-gobodo-db-stg (postgres), gobodo-* S3 | Custom objects, Client__c, PageLayoutTemplate__c; **PubliclyAccessible** (SG 0.0.0.0/0 on 5432) |
| DynamoDB | (none enumerated) | Read-optimized projections / fast dashboards |
| GitHub | — | Code/config as deployable behavior |

### Vendor Systems

| Vendor | Invoked By | Creds Location |
|--------|------------|----------------|
| Array | core-cdk-lambda-array-*, creditblock-cdk-lambda-*, vendor-array-cdk-statemachine-* | ParameterStore:/cybersweep-serverless/array/* |
| CyberSweep | cybersweep-cdk-lambda-*, cybersweep-cdk-statemachine-* | ParameterStore:/cybersweep-serverless/* |
| Vouched | core-cdk-lambda-vouched-* | SecretsManager |
| Stripe | Product Hub, Invoicing | SecretsManager |

### Front Ends

| App | AWS / Domain | Status |
|-----|--------------|--------|
| Client Dashboard | partner.dev.regalcredit.com, partner.stg.regalcredit.com, partner.release.regalcredit.com (CloudFront) | Deployed |
| Product Hub | Local/Staging; Salesforce-style CRM surface | In Development |
| B2C apps | RevolutxRCMB2Cios, RegalCreditB2Cios | Deployed |

---

## 4) API Surface (Deployed Endpoints)

| Domain | API ID | Stage | Path Patterns | Lambda Domain |
|--------|--------|-------|---------------|---------------|
| api.dev.regalcredit.com | yqo32wi8cd (rcm-cdk-apigateway-development-pl) | development | /core/*, /creditblock/*, /cybersweep/*, /vendor/* | core/creditblock/cybersweep/vendor-cdk-lambda-*-development-pl |
| api.stg.regalcredit.com | fl1928grye (rcm-cdk-apigateway-staging-pl) | staging | (same) | *-staging-pl |
| api.release.regalcredit.com | gy0zwkmroe (rcm-cdk-apigateway-production-pl) | production | (same) | *-production-pl |
| webhook.release.regalcredit.com | ndcltu5vvd (rcm-cdk-webhook-apigateway-production-pl) | production | /array/listener, /array/pip/callback/*, /vouched | core-cdk-lambda-array-webhook-*, vendor-cdk-lambda-pip-webhook-router-*, core-cdk-lambda-vouched-webhook-* |

**Path → Lambda mapping:**
- `/core/v1/partner/*` → core-cdk-lambda-contact-*
- `/core/v1/client/*` → core-cdk-lambda-client-*
- `/core/v1/array/*` → core-cdk-lambda-array-*
- `/core/v1/verified/*` → core-cdk-lambda-vouched-*
- `/creditblock/*` → creditblock-cdk-lambda-bureau-*
- `/cybersweep/*` → cybersweep-cdk-lambda-*
- `/vendor/array/*` → vendor-cdk-lambda-credit-report-*, vendor-cdk-lambda-pip-*

---

## 5) Core Entities (Full Catalog)

### A) Identity / Access Entities

| Entity | Gobodo API | RDS | Description |
|--------|------------|-----|-------------|
| UserIdentity | — | — | Authenticated identity (Keycloak user); links to Client or Contact |
| Device | — | — | deviceId, userId, deviceType, deviceToken; lifecycle: register → update token → retire |

### B) Customer / Relationship Entities

| Entity | Gobodo API | RDS | Description |
|--------|------------|-----|-------------|
| Client | Client__c | ✓ | Person Account; primary customer. `accountId` links to Account. |
| Account | Account__c | ✓ | Firm (Partner) or Individual. Type field distinguishes. |
| Contact | Contact__c | ✓ | Person at a firm (PartnerContact); `can see clients` via relationships |
| Lead | Lead__c | ✓ | Prospective client; sales pipeline entry |
| Opportunity | Opportunity__c | ✓ | Sales opportunity; links to Lead/Account |
| FamilyMember | — | — | Spouse \| Dependent; sub-client for eligibility/pricing. **Future/Planned** |
| ClientVisibilityLink | — | — | Many-to-many Contact ↔ Client; implicit via relationships |

### C) Service / Product Entities

| Entity | Gobodo API | RDS | Description |
|--------|------------|-----|-------------|
| Product | Product | ✓ | Service offering catalog (via Stripe/Gobodo) |
| Pricebook | Pricebook__c | — | Partner-specific pricing; `accountId`, `partnerId` |
| ClientPriceOverride | — | — | Per-client pricing exception |
| Contract | Contract__c | — | Service agreement |
| Invoice | — (Stripe) | — | Billing record (Stripe Invoice) |
| Enrollment | — | — | Client ↔ Service binding; embedded in module entities (e.g., `enrollmentDate`) |

**Services (logical, not first-class objects):**
- Credit Block
- Credit Monitoring
- Credit Repair
- CyberSweep (Identity Protection)

### D) Workflow / Operations Entities

| Entity | Gobodo API | RDS | Description |
|--------|------------|-----|-------------|
| Task | Task__c | ✓ | Action item / follow-up |
| Event | Event__c | ✓ | Calendar/scheduled event |
| Note | Note__c | — | Free-form annotation |
| Report | Report__c | — | Analytics/compliance report |
| SupportCase | Case__c | ✓ | Customer support ticket |
| Notification | — | — | In-app alert; type, level, read receipt |
| Verification | — | — | Identity verification (Vouched); /core/v1/verified/* |

### E) Credit Repair Entities (Array)

| Entity | Gobodo API | RDS | Description |
|--------|------------|-----|-------------|
| BureauCase | — | — | Block/unblock request; /creditblock/v1/bureaus/* |
| ArrayUser | — | — | Client's Array vendor identity; /vendor/array/v2/user |
| ArrayMonitoringEnrollment | — | — | Active monitoring; /vendor/array/v1/monitoring/enrollments |
| CreditReport | — | — | Credit report; /vendor/array/v2/report/order, /pdf |
| WebhookEvent | — | — | POST /array/listener (service, method, event, details) |

### F) Identity Protection Entities (CyberSweep)

| Entity | Gobodo API | RDS | Description |
|--------|------------|-----|-------------|
| CyberSweepClient | — | — | Client's CyberSweep enrollment; includes `enrollmentDate`, `monitoringStatus` |
| CyberSweepMonitor | — | — | Active identity monitor; /cybersweep/v1/idp/monitors/* |
| CSAudit | — | — | CyberSweep audit/scan result |
| PAWRecord | — | — | PII/audit record (Personal Audit Workflow) |

---

## 6) Relationships (Graph Edges)

### Client Relationships
```
Client HAS_PROFILE ClientProfile
Client HAS_FAMILY_MEMBER FamilyMember (planned)
Client HAS_DEVICE Device
Client HAS_NOTIFICATIONS Notification
Client HAS_SUPPORT_CASE SupportCase
Client HAS_VERIFICATION VouchedVerification
Client HAS_BUREAU_CASE BureauCase
Client HAS_ARRAY_USER ArrayUser
Client HAS_CYBERSWEEP_MONITOR CyberSweepMonitor
Client HAS_TASKS Task
Client HAS_EVENTS Event
Client HAS_NOTES Note
```

### Partner/Account Relationships
```
Account (Partner) HAS_PRICEBOOK Pricebook
Account (Partner) HAS_CONTACT Contact
Contact CAN_VIEW Client (legacy + many-to-many)
Account HAS_CONTRACTS Contract
Account HAS_OPPORTUNITIES Opportunity
```

### Vendor Relationships
```
ArrayUser HAS_MONITORING_ENROLLMENTS ArrayMonitoringEnrollment
ArrayUser HAS_CREDIT_REPORTS CreditReport
CyberSweepClient HAS_MONITORS CyberSweepMonitor
CyberSweepMonitor HAS_AUDITS CSAudit
```

### Event Relationships
```
WebhookEvent GENERATES Notification
WebhookEvent UPDATES Salesforce
Lead CONVERTS_TO Client
Lead CONVERTS_TO Opportunity
Opportunity CONVERTS_TO Contract
```

---

## 7) Object API Names (Quick Reference)

| App Object | Gobodo API Name | RDS | Domain |
|------------|-----------------|-----|--------|
| Client | Client__c | ✓ | Credit Development |
| Contact | Contact__c | ✓ | Credit Development |
| Lead | Lead__c | ✓ | Credit Development |
| Opportunity | Opportunity__c | ✓ | Credit Development |
| Account | Account__c | ✓ | Credit Development |
| Product | Product | ✓ | Credit Development |
| Pricebook | Pricebook__c | — | Credit Development |
| Contract | Contract__c | — | Credit Development |
| Invoice | (Stripe) | — | Credit Development |
| Case | Case__c | ✓ | Credit Repair |
| Task | Task__c | ✓ | Credit Repair |
| Event | Event__c | ✓ | Credit Repair |
| Note | Note__c | — | Credit Repair |
| Report | Report__c | — | Identity Protection |

---

## 8) Canonical Events (Agent Triggers)

| Domain | Category | Events |
|--------|----------|--------|
| **All** | Identity | device_registered, device_token_updated |
| **Credit Development** | Client | client_profile_updated, family_member_updated |
| **Credit Development** | Partner | partner_contact_created/updated, client_visibility_changed |
| **Credit Development** | Sales | lead_created, lead_converted, opportunity_created, opportunity_won/lost |
| **Credit Development** | Billing | invoice_created, invoice_paid, invoice_overdue |
| **All** | Notifications | notification_created, notification_read_receipt_updated |
| **Credit Repair** | Support | support_case_created, support_case_resolved |
| **Credit Repair** | Tasks | task_created, task_completed, task_overdue |
| **Credit Repair** | Verification | verification_created, verification_case_created |
| **Credit Repair** | Credit Block | bureau_block_requested, bureau_unblock_requested, bureau_check_status_requested, bureau_summary_updated, block_process_completed |
| **Credit Repair** | Array | array_user_created, array_user_authenticated, array_monitoring_enrollment_submitted/completed, report_ordered, report_pdf_ready, array_webhook_received |
| **Identity Protection** | CyberSweep | cybersweep_monitors_listed, cybersweep_monitor_added/updated/deleted, cybersweep_queue_automation_submitted |
| **Identity Protection** | Audits | cs_audit_completed, paw_record_created |
| **All** | Data Sync | rcm_dsync_sync_scheduled (cron 07:00 UTC), rcm_dsync_sync_completed, rcm_dsync_sync_failed |

---

## 9) Rules → Agent Actions

| Domain | Trigger | Preconditions | Action | Side Effects |
|--------|---------|---------------|--------|--------------|
| Identity Protection | Webhook alertType indicates new exposure | authorization + linked records | Recommend CyberSweep + Monitoring expansion | Notify client manager |
| Credit Repair | Report PDF 202→204 failure | — | Create support case draft + ops task + retry | — |
| Credit Repair | checkStatusResult token resolved | — | Update bureau summary + notification | — |
| Identity Protection | Cybersweep monitor fails (missing arrayClientKey) | — | Open ops task: "missing vendor mapping" | Recommend register user / fetch key |
| Credit Development | Lead inactive >30 days | — | Create follow-up task | Notify assigned rep |
| Credit Development | Opportunity stage changed | — | Update forecast + notify manager | — |

---

## 10) Permissions Model

### Keycloak Scopes (Capabilities)
- `salesforce:read` vs `salesforce:read-write`
- `client-dashboard:read` vs `client-dashboard:read-write`

### CASL Subjects (Product Hub)
Client, Case, User, Contact, Lead, Opportunity, Event, Task, Report, Account, Product, Pricebook

### CASL Actions
read, write, delete, search, access, manage

### Verification Requirements
Actions requiring verified identity (Vouched):
- Bureau block/unblock
- Verification submission
- Vendor enrollments

Agents must know: **can I do this, or should I only draft + ask approval?**

---

## 11) Deployment Topology (Repo → AWS)

### Accounts & Regions

| Account | Envs | Regions |
|---------|------|---------|
| 865942198740 | dev, staging | us-east-1, us-east-2 |
| 975050223773 | release, production | us-east-1, us-east-2 |

### Bounded Contexts → Repos → AWS Stacks

| Domain | Repos | AWS Stacks / Resources |
|--------|-------|------------------------|
| Identity & Access | keycloak-v26, keycloak-theme-partner-portal | keycloak-development, keycloak-v16-release |
| Client & Partner | rcm-serverless, client-dashboard-api-v2, RegalCreditB2BWebDashboard | rcm-pipeline-rcm-stack-*, rcm-pipeline-core-stack-*, client-dashboard-pipeline-stack-* |
| Enrollment & Onboarding | classic-registration-forms, Regal-Product-Hub, rcm-serverless | regal-credit-api-dev/stg |
| CreditBlock | cb-submodule, rcm-serverless | rcm-pipeline-creditblock-stack-*, creditblock-cdk-lambda-* |
| Monitoring / Array | vendor-submodule, rcm-serverless | rcm-pipeline-vendor-stack-*, vendor-cdk-lambda-*, core-cdk-statemachine-array-* |
| CyberSweep | cs-submodule, rcm-serverless | rcm-pipeline-cybersweep-stack-*, cybersweep-cdk-lambda-* |
| Data Sync & Projections | salesforce-data-sync (rcm-dsync) | rcm-dsync-sync-orchestrator-*, rcm-dsync-sync-dispatcher-*, rcm-dsync-compute-*-* |
| Comms & Support | rcm-email-templating-engine, rcm-email-classifier-service, task-webhook-service | (not in AWS inventory) |
| Gobodo | gobodo-infrastructure | gobodo-infrastructure-rds-*, gobodo-cache-* |
| Product Hub | Regal-Product-Hub | Local/Staging (not yet deployed to AWS) |

### rcm-dsync Footprint (Salesforce → RDS)

| Component | Dev/Staging (865942198740) | Production (975050223773) |
|-----------|---------------------------|---------------------------|
| Step Functions | rcm-dsync-sync-orchestrator-*, rcm-dsync-sync-dispatcher-* | **None** |
| EventBridge | rcm-dsync-sync-schedule-* (cron 07:00 UTC) | **None** |
| Lambdas | rcm-dsync-compute-*-init-sync, fetch-batch, persist-worker, finalize-sync | **None** |
| RDS | rcm-dsync-data-main-rds-stg + Proxy | **None** |
| SQS | rcm-dsync-queue-stg-persist, *-eventbridge-dlq | **None** |
| SNS | rcm-dsync-monitoring-stg-alarms | **None** |

**Orchestrator flow:** InitSync → FetchBatch → EnqueueBatches (SQS) → FinalizeSync; NotifyFailure → SNS.  
**Dispatcher:** Map over object_types → StartSyncExecution (orchestrator per type).

---

## 12) Data Flow Summary

### Primary Data Sources
- **RDS:** When `USE_RDS_FOR_CORE_OBJECTS` and `RDS_API_URL` set; reads from RDS API
- **Gobodo:** Default for CRM objects; `normalizeToGobodoFormat()` maps to `*__c` names
- **Salesforce:** Legacy source; synced to RDS via rcm-dsync
- **Mock/localStorage:** Fallback when neither API is available

### Sync Flow
```
Salesforce → rcm-dsync (07:00 UTC daily) → RDS
                                            ↓
Product Hub reads from: RDS (preferred) OR Gobodo (default)
Product Hub writes to: Gobodo
```

### RDS Core Objects
Account, Contact, Lead, Opportunity, Client, Product, Partner, Case, Task, Event

---

## 13) Alignment Notes (Ontology ↔ Product Hub)

### Naming Conventions
| Ontology Term | Product Hub Term | Notes |
|---------------|------------------|-------|
| Partner | Account (type: firm) | Ontology "Partner" = app "Account" with firm type |
| PartnerContact | Contact | Contact at a firm |
| Client | Client (Person Account) | Primary entity; full alignment |
| Service | Module/Feature | Not first-class objects; implemented as features |
| Enrollment | `enrollmentDate` field | Embedded in module entities, not separate object |

### Planned/Future Entities
- **FamilyMember:** Defined in ontology, not yet in Product Hub
- **ClientVisibilityLink:** Implicit via relationships, consider explicit junction object

### Gap Summary
| Status | Entity |
|--------|--------|
| ✅ Aligned | Client, Contact, Account, Lead, Opportunity, Case, Task, Event, Product, Pricebook |
| ⚠️ Naming Mismatch | Partner (ontology) = Account/firm (app) |
| ⚠️ Embedded | Enrollment (ontology) = enrollmentDate (app) |
| 🔜 Planned | FamilyMember, ClientVisibilityLink |
| 📱 App-Only | Report, Note, Contract, Invoice, ClientPriceOverride |
| 🔌 Vendor-Specific | CyberSweepClient, CSAudit, PAWRecord |

---

## Appendix A: Agency & Personal Ontology (Reference)

**Agency:** Clients have contracts; campaigns interact with APIs; agents monitor metrics and adjust campaigns.  
**Personal:** Goals → routines; agents track habits and nudge adjustments.

---

## Appendix B: Changelog

| Version | Date | Changes |
|---------|------|---------|
| 2.0 | 2026-02-21 | Initial v2 from AWS inventory + ontology v1 |
| 2.1 | 2026-02-23 | Added Business Domains (Credit Development, Credit Repair, Identity Protection); Added Product Hub entities (Lead, Opportunity, Task, Event, Report, Note, Product, Invoice, ClientPriceOverride, Contract, CyberSweepClient, CSAudit, PAWRecord); Object API reference table; Alignment notes; Data flow summary |
