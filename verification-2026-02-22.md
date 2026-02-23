# AWS Verification Scan — 2026-02-22

**Account:** 865942198740 (Development)
**Profile:** regal-dev
**Scanned by:** Optimus

---

## Summary

| Resource | us-east-1 | us-east-2 | Notes |
|----------|-----------|-----------|-------|
| Lambdas | 285 | 42 | More than expected |
| RDS Instances | 8 | - | Gobodo PUBLIC! |
| API Gateways | 30 | - | Many test instances |
| Amplify Apps | 6 | - | Matches ontology |
| CloudFormation Stacks | 34 | - | Good coverage |

---

## 🚨 SECURITY ISSUES CONFIRMED

### Gobodo DB Publicly Accessible
- **Instance:** `rcm-gobodo-db-stg`
- **Endpoint:** `rcm-gobodo-db-stg.cqhna2fiukih.us-east-1.rds.amazonaws.com`
- **Security Group:** `sg-09f4dfb35576fa76a`
- **Port 5432:** Open to `0.0.0.0/0` ⚠️

**Recommendation:** Immediate remediation required. Restrict to VPC CIDR.

---

## RDS Instances

| Instance | Engine | Status | Public |
|----------|--------|--------|--------|
| keycloak-development | aurora-mysql | available | ❌ No |
| keycloak-development-...-instance-1 | aurora-mysql | available | ❌ No |
| rcm-dsync-data-main-rds-dev | postgres | available | ❌ No |
| rcm-dsync-data-main-rds-dev-alex | postgres | available | ❌ No |
| rcm-dsync-data-main-rds-dev-jim | postgres | available | ❌ No |
| rcm-dsync-data-main-rds-dev-wei | postgres | available | ❌ No |
| rcm-dsync-data-main-rds-stg | postgres | available | ❌ No |
| **rcm-gobodo-db-stg** | postgres | available | **⚠️ YES** |

---

## API Gateways (30 total)

### Production-like
- `fl1928grye` — rcm-cdk-apigateway-staging-pl
- `yqo32wi8cd` — rcm-cdk-apigateway-development-pl (matches ontology!)

### By Domain
- **core-cdk-apigateway-*** — 6 instances (dev, staging, test-alex, test-wei)
- **creditblock-cdk-apigateway-*** — 3 instances
- **cybersweep-cdk-apigateway-*** — 4 instances
- **vendor-cdk-apigateway-*** — 4 instances
- **rcm-cdk-apigateway-*** — 4 instances
- **gobodo-cache-infra-dev** — 1 instance
- **webhook APIs** — 6 instances

---

## Amplify Apps

| App Name | Domain |
|----------|--------|
| partnerportal2023 | d1f7cg9rxlwckc.amplifyapp.com |
| Regal-Product-Hub-Stg | d2r0codbnevnxq.amplifyapp.com |
| next-js-amplify-deploy | d3d63du486qvxu.amplifyapp.com |
| RegalCreditB2BWebDashboard | d3h9qkfgty36hl.amplifyapp.com |
| Regal-Product-Hub-Dev | di8ftbeldh0qk.amplifyapp.com |
| RegalCreditB2BWebDas | dmcsbvlvtl0c7.amplifyapp.com |

**Note:** Product Hub apps exist (Dev + Stg) — matches ontology!

---

## CloudFormation Stacks (34)

### Pipeline Stacks
- client-dashboard-pipeline-stack-development
- client-dashboard-pipeline-stack-staging
- rcm-pipeline-core-stack-staging
- rcm-pipeline-creditblock-stack-development/staging
- rcm-pipeline-cybersweep-stack-development/staging
- rcm-pipeline-vendor-stack-development/staging

### Infrastructure Stacks
- gobodo-cache-infra-dev
- gobodo-cache-pipeline-dev/stg
- gobodo-eb-gobodo-staging
- gobodo-infrastructure-rds-rcm-gobodo-db-stg
- gobodo-infrastructure-s3
- rcm-gobodo-neo4j-stg

### CDK Stacks
- rcm-core-cdk-stack-development
- rcm-creditblock-cdk-stack-development
- rcm-cybersweep-cdk-stack-development

### VPCs
- regal-dev-vpc
- regal-stg-vpc

---

## Lambda Functions (285 in us-east-1)

### By Prefix Count
- `core-cdk-lambda-*` — ~100+ functions
- `creditblock-cdk-lambda-*` — ~30 functions
- `cybersweep-cdk-lambda-*` — ~40 functions
- `vendor-cdk-lambda-*` — ~30 functions
- `gobodo-*` — ~5 functions
- `array*` — ~10 legacy functions
- `amplify-*` — ~4 Cognito triggers

### Sample Functions (matches ontology patterns)
- `core-cdk-lambda-account-*`
- `core-cdk-lambda-array-*`
- `core-cdk-lambda-client-*`
- `core-cdk-lambda-contact-*`
- `creditblock-cdk-lambda-bureau-*`
- `cybersweep-cdk-lambda-*`
- `vendor-cdk-lambda-*`

---

## Ontology Validation Results

### ✅ Confirmed
- [x] Keycloak Aurora MySQL databases exist
- [x] RDS sync databases exist (dev, stg, per-developer)
- [x] Gobodo DB exists (but PUBLIC - security issue)
- [x] API Gateway IDs match for dev (`yqo32wi8cd`)
- [x] Amplify apps for Product Hub exist (dev + stg)
- [x] Lambda naming convention matches ontology
- [x] CloudFormation stack patterns match

### ⚠️ Discrepancies / Needs Update
- [ ] Ontology says `fl1928grye` is staging API - CORRECT
- [ ] Production API IDs need verification (different account)
- [ ] More Lambda functions than documented
- [ ] Neo4j stack exists (`rcm-gobodo-neo4j-stg`) - not in ontology
- [ ] Multiple developer-specific RDS instances (alex, jim, wei)

### ❓ Need Production Account Scan
- Production API Gateway IDs
- Production RDS instances
- Production Lambda count

---

## Next Steps

1. **URGENT:** Fix Gobodo security group
2. Scan production account (975050223773)
3. Update ontology with discovered resources
4. Document Neo4j usage
5. Verify production URLs

---

## Production Account Scan (975050223773)

### RDS Instances (6 total)
All Keycloak Aurora MySQL, all NOT public ✅

| Instance | Engine | Public |
|----------|--------|--------|
| keycloak-release-...-4-instance-1 | aurora-mysql | ❌ No |
| keycloak-release-...-e-instance-1 | aurora-mysql | ❌ No |
| keycloak-release-...-k-instance-1 | aurora-mysql | ❌ No |
| keycloak-release-...-t-instance-1 | aurora-mysql | ❌ No |
| keycloak-release-...-t-instance-2 | aurora-mysql | ❌ No |
| keycloak-v16-release-rdsdbinstance-* | aurora-mysql | ❌ No |

**⚠️ NO rcm-dsync RDS in production!**

### API Gateways (14 total)

| ID | Name | Notes |
|----|------|-------|
| `gy0zwkmroe` | rcm-cdk-apigateway-production-pl | ✅ Matches ontology! |
| `ndcltu5vvd` | rcm-cdk-webhook-apigateway-production-pl | ✅ Matches ontology! |
| e0b1zp7bw9 | core-cdk-apigateway-release-pl | |
| 5ikno6dq92 | creditblock-cdk-apigateway-release-pl | |
| p1d8hit5r5 | cybersweep-cdk-apigateway-release-pl | |
| s96p77o4j4 | vendor-cdk-apigateway-release-pl | |
| 3mplyar71f | core-cdk-webhook-apigateway-release-pl | |
| 07cpn3rj29 | rcm-cdk-webhook-apigateway-release-pl | |
| + others | test-magnus instances | |

### Amplify Apps
**None in production account!**
- Client Dashboard production hosting unclear
- May use different account or service

### CloudFormation Stacks (21 total)
- `keycloak-v16-release`
- `rcm-core-cdk-stack-production`
- `rcm-creditblock-cdk-stack-production`
- `rcm-cybersweep-cdk-stack-production`
- `rcm-pipeline-*-stack-production/release` (multiple)
- `client-dashboard-pipeline-stack-production`

### Key Production Findings
1. ✅ API Gateway IDs match ontology perfectly
2. ✅ Keycloak Aurora running, not public
3. ⚠️ No rcm-dsync (Salesforce sync) in prod
4. ⚠️ No Amplify apps in prod account
5. ⚠️ No Gobodo in prod account

---

## Ontology Updated

The following sections were updated with verified AWS data:
- Section 3.1: Identity (Keycloak instances verified)
- Section 3.2: Core Data (RDS instances, dsync status)
- Section 3.3: Gobodo (security issue details, Neo4j discovered)
- Section 3.5: Front Ends (Amplify apps verified)
- Section 4.1: API Gateways (IDs verified)
- Section 10: Migration (confirmed no prod resources)

---

*Scan completed: 2026-02-22 18:45 PST*
*Production scan added: 2026-02-22 18:48 PST*
