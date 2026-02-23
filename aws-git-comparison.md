# AWS ↔ Git Cross-Reference
**Generated:** 2026-02-22

## Summary

| AWS Component | Git Repository | Status |
|---------------|----------------|--------|
| `core-cdk-lambda-*` | rcm-serverless + core-submodule | ✅ Active |
| `creditblock-cdk-lambda-*` | rcm-serverless + cb-submodule | ✅ Active |
| `cybersweep-cdk-lambda-*` | rcm-serverless + cs-submodule | ✅ Active |
| `vendor-cdk-lambda-*` | rcm-serverless + vendor-submodule | ✅ Active |
| `rcm-dsync-*` | salesforce-data-sync | ✅ Active |
| `gobodo-*` | ❓ Unknown | 🔍 Need to find |
| Keycloak (Aurora) | keycloak-v26, keycloak-theme-partner-portal | ✅ Active |
| Regal-Product-Hub (Amplify) | Regal-Product-Hub | ✅ Active |
| RegalCreditB2BWebDashboard (Amplify) | RegalCreditB2BWebDashboard | ✅ Active |
| client-dashboard-api-v2 | client-dashboard-api-v2 | ✅ New |
| classic-registration-forms | classic-registration-forms | ✅ Enrollment pages |

---

## Detailed Mapping

### 1. Serverless Lambdas (rcm-serverless)
**Repo:** `Regal-Credit-Management/rcm-serverless`
**Last commit:** 2026-02-03 (v1.7.4)

| AWS Lambda Pattern | Submodule | Count (Dev) |
|--------------------|-----------|-------------|
| `core-cdk-lambda-*` | core-submodule | ~100+ |
| `creditblock-cdk-lambda-*` | cb-submodule | ~30 |
| `cybersweep-cdk-lambda-*` | cs-submodule | ~40 |
| `vendor-cdk-lambda-*` | vendor-submodule | ~30 |

**Total Lambdas:** 285 (us-east-1) + 42 (us-east-2) = 327

### 2. Salesforce Data Sync (salesforce-data-sync)
**Repo:** `Regal-Credit-Management/salesforce-data-sync`
**Last commit:** 2026-02-18

| AWS Resource | Instance/Name |
|--------------|---------------|
| RDS (dev) | rcm-dsync-data-main-rds-dev |
| RDS (stg) | rcm-dsync-data-main-rds-stg |
| RDS (dev-alex) | rcm-dsync-data-main-rds-dev-alex |
| RDS (dev-jim) | rcm-dsync-data-main-rds-dev-jim |
| RDS (dev-wei) | rcm-dsync-data-main-rds-dev-wei |
| Step Functions | rcm-dsync-sync-orchestrator-* |
| EventBridge | rcm-dsync-sync-schedule-* |
| Lambdas | rcm-dsync-compute-* |

**⚠️ Production:** NO rcm-dsync resources in prod account (975050223773)

### 3. Regal Product Hub
**Repo:** `Regal-Credit-Management/Regal-Product-Hub`
**Last commit:** 2026-02-20 by magnusregal (Gobodo sync PR!)

| AWS Resource | Domain |
|--------------|--------|
| Amplify (dev) | Regal-Product-Hub-Dev → di8ftbeldh0qk.amplifyapp.com |
| Amplify (stg) | Regal-Product-Hub-Stg → d2r0codbnevnxq.amplifyapp.com |
| Amplify (prod) | ❌ Not deployed |

### 4. Client Dashboard (B2B)
**Repo:** `Regal-Credit-Management/RegalCreditB2BWebDashboard`
**Last commit:** 2026-01-27 by magnusregal

| AWS Resource | Domain |
|--------------|--------|
| Amplify | RegalCreditB2BWebDashboard → d3h9qkfgty36hl.amplifyapp.com |
| CloudFormation | client-dashboard-pipeline-stack-* |

**Also:**
- `client-dashboard-api-v2` (new API, 2026-02-10)
- `client-dashboard` (older version?)
- `b2b-web-dashboard` (older version?)

### 5. Keycloak (Identity)
**Repos:**
- `keycloak-v26` (Last: 2025-01-23)
- `keycloak-theme-partner-portal` (Last: 2026-01-29)

| AWS Resource | Account | Notes |
|--------------|---------|-------|
| keycloak-development (Aurora) | Dev | 2 instances |
| keycloak-v16-release (Aurora) | Prod | 6 instances |
| CloudFormation: keycloak-v16-release | Prod | |

### 6. Gobodo ❓
**AWS Resources Found:**
- `rcm-gobodo-db-stg` (RDS Postgres) — **PUBLICLY ACCESSIBLE!**
- `rcm-gobodo-neo4j-stg` (CloudFormation)
- `gobodo-cache-infra-dev` (API Gateway)
- `gobodo-eb-gobodo-staging` (Elastic Beanstalk?)
- `GobodoCacheLambda-Dev`, `CacheManagementLambda-Dev`, `StoreCacheLambda-Dev`

**Git Repos:** ❓ None found with "gobodo" in name
- Could be external vendor code
- Could be in a different org/private repo
- **ACTION:** Ask team where Gobodo source lives

### 7. Enrollment/Registration
**Repo:** `classic-registration-forms` (Last: 2026-01-29)

Not on Amplify — likely CloudFront + S3 or different hosting

### 8. Mobile Apps
**Repos:**
- `RegalCreditB2Cios` (Last: 2025-10-20)
- `RevolutxRCMB2Cios` (Last: 2025-10-20)

iOS native apps — deployed via App Store/TestFlight

---

## Repos Without Clear AWS Mapping

| Repo | Last Updated | Purpose |
|------|--------------|---------|
| Data-Analytics | 2024-12-18 | Analytics? |
| Issue-Reporting | 2025-08-12 | Internal tooling? |
| Mock-Bureaus | 2025-01-21 | Testing credit bureau mocks |
| Project-Management | 2025-03-25 | PM docs? |
| Reports-and-Analytics | 2025-03-20 | Reporting? |
| rcm-email-classifier-service | 2026-02-10 | Email processing |
| rcm-email-templating-engine | 2026-02-10 | Email templates |
| rcm-file-mapper | 2025-11-05 | File processing |
| rcm-n8n | 2025-08-21 | n8n workflows |
| task-webhook-service | 2026-02-10 | Webhook handling |
| genealogy-platform | 2026-01-11 | ❓ |

---

## AWS Resources Without Clear Git Mapping

| AWS Resource | Notes |
|--------------|-------|
| Gobodo (all) | Source repo unknown |
| `amplify-login-*` Lambdas | Cognito triggers (auto-generated?) |
| `array*` Lambdas | Legacy Array integration code |
| Various test stacks | alex, jim, wei, magnus test instances |

---

## Submodule Structure

The `rcm-serverless` repo uses submodules for domain separation:

```
rcm-serverless/
├── core-submodule/      → core-cdk-lambda-*
├── cb-submodule/        → creditblock-cdk-lambda-*
├── cs-submodule/        → cybersweep-cdk-lambda-*
├── vendor-submodule/    → vendor-cdk-lambda-*
├── utils-submodule/     → shared utilities
└── test-submodule/      → test helpers
```

---

## Questions to Resolve

1. **Where is Gobodo source code?** (Not in Regal-Credit-Management org)
2. **Why no rcm-dsync in production?** (Blocker? Timeline?)
3. **What is genealogy-platform?** (New product?)
4. **client-dashboard vs client-dashboard-api-v2 vs b2b-web-dashboard?** (Evolution or parallel?)
5. **Array legacy lambdas** — still in use or deprecated?

---

## Recommendations for Ontology

1. ✅ Add GitHub org name: `Regal-Credit-Management`
2. ✅ Document submodule structure
3. ⚠️ Flag Gobodo source location as TBD
4. ⚠️ Add email services (classifier, templating)
5. ⚠️ Add webhook service
6. ⚠️ Document n8n usage

---

*Generated: 2026-02-22 18:50 PST*
