---
name: AEM Project Overview
description: High-level overview of the Dell AEM learning portal project (devstore) — purpose, modules, integrations, CI/CD
type: project
originSessionId: 62767ced-1e42-4928-a161-34e6b010f8e3
---
**Project**: Dell Technologies Learning Hub (AEM Devstore)  
**Artifact**: devstore, version 1.0.0-SNAPSHOT  
**Purpose**: AEM-based education services portal for Dell Technologies training — users discover, evaluate, and register for courses (classroom, virtual, on-demand, certifications, packages).

**Why:** Enterprise learning portal for Dell's customers/partners, integrating multiple external systems for course aggregation, search, auth, and purchases.  
**How to apply:** Frame all component/service work in the context of a training catalog UX with multi-currency, multi-locale, and partner-tier support.

## Maven Module Structure

| Module | Purpose |
|--------|---------|
| core | Java OSGi bundle — services, models, servlets, filters, REST clients, beans |
| ui.apps | AEM HTL components, dialogs, clientlibs (JS/CSS) |
| ui.config | OSGi configurations per environment |
| common.core | Minimal shared utility module |
| all | Deployment aggregator (produces devstore.all-1.0.0-SNAPSHOT.zip) |

- `ui.frontend` module is commented out/disabled
- Java 1.8, AEM 6.5.17, Core WCM Components 2.21.2
- Node v16.17.0 / npm 8.15.0 for frontend

## Key Integrations

- **Coveo Search** — org IDs `dellsandbox2` (dev) / `dellprod` (prod), SearchHub `EducationHub`, faceted search + analytics
- **Thingworx** (OneLMS.CourseAggregatorServiceManager) — course aggregation API at `ptcappstg02.isus.emc.com:8080`
- **SAML 2.0** — enterprise SSO with redirect/callback/logout/response-processor servlets
- **PRM (Partner Relationship Management)** — Sharepoint integration for partner content
- **Voucher System** — training voucher generation and management
- **Layer7** — API gateway/security layer
- **Email Service** — notifications/confirmations
- **Tech Direct** — technical documentation API

## CI/CD (.gitlab-ci.yml)

- Build image: `harbor.dell.com/devops-images/debian-12/java-8-jdk:latest`
- Stages: compile → publish (Artifactory via HashiCorp Vault) → unit-tests / malware-scan / inclusive-lang → deploy per env → RFC → DAST
- Environments: DEV (auto on feature branch), STAGE/GE2 (manual), PREPROD/SIT (manual), PROD (manual + RFC approval)
- Target artifact: `all/target/devstore.all-1.0.0-SNAPSHOT.zip`
- Secrets via HashiCorp Vault
