# Consolidated Assessment Report

Code Assessment Summary

**Assessment Date:** November 23, 2026  
**Components Assessed:** [1](#component-assessment-matrix)

## Dashboard

High-level consolidation assessment of application health and cloud readiness across tech stacks.

| Metric | Value |
|--------|-------|
| Total Components | [1](#component-assessment-matrix) |
| Upgrades Needed | [1](#component-assessment-matrix) |
| Mandatory Blockers | 1 |
| Potential Issues | 2 |

### Technology Distribution

| Framework | Components |
|-----------|-----------|
| .NET Framework 4.8 | [1](#component-assessment-matrix) |

### Effort and Resource Summary

| Effort | Components |
|--------|-----------|
| S | [0](#component-assessment-matrix) |
| M | [1](#component-assessment-matrix) |
| L | [0](#component-assessment-matrix) |
| XL | [0](#component-assessment-matrix) |

## Recommendations

Key architectural decisions and recommended migration sequencing.

> **Important:** These architectural recommendations represent key decision points required to move forward with migration. Review them now to avoid delays.

### Recommended Azure Services

| Dependency | Components | Recommendation | Rationale |
|------------|-----------|----------------|-----------|
| Local File System | 1 | Azure Blob Storage | Azure Blob Storage replaces local file system dependencies with scalable, geo-redundant cloud storage accessible from any Azure service. |
| Plaintext Credentials | 1 | Azure Key Vault with Managed Identity | Storing credentials in Azure Key Vault with managed identity eliminates plaintext secrets from code and configuration, improving security posture. |
| SQL Server | 1 | Azure SQL Database | Azure SQL Database provides a fully managed SQL Server experience with built-in high availability, automatic backups, and managed identity authentication. |
| Windows AD | 1 | Microsoft Entra ID | Microsoft Entra ID provides cloud-native identity and access management, replacing on-premises Windows AD with modern authentication protocols. |
| Compute | 1 | App Service MI | Target compute platform for application hosting. |

### Recommended Target Platform

| Recommendation | Components | Rationale |
|----------------|-----------|-----------|
| App Service MI | 1 | Legacy .NET Framework application recommended for App Service with Managed Instance for Windows compatibility. |

### Recommended Upgrade Path

| Framework | Components | Recommendation | Rationale |
|-----------|-----------|----------------|-----------|
| .NET Framework 4.8 | 1 | Migrate to .NET 10 | .NET 10 is the latest LTS version supported by GitHub Copilot Modernization, enabling automated upgrades with minimal manual effort. |

### Migration Wave Plan

| Phase | Business Apps | Rationale |
|-------|-------------|-----------|
| Wave 1 - Quick Wins | - | Use the smallest in-scope services to define deployment and validation patterns before larger workstreams. |
| Wave 2 - Core Modernization | shared-legacy-app | Include integration refactoring as a contained modernization workstream. |
| Wave 3 - Complex Upgrades | - | Plan after shared patterns are established; this wave carries the largest upgrade, validation, and packaging scope. |

### 6R Recommendation

| 6R | Components | Rationale |
|----|------------|-----------|
| Replatform | 1: BookCatalog.Web | |

## Business App Assessment Matrix

Detailed framework upgrade plan with readiness assessment and migration requirements.

> **Legend:** M = Mandatory, P = Potential, O = Optional

| Business App | Repository | Component | Framework | Recommended Target Platform | Upgrade Recommendation | Issues (M/P/O) | Effort | Decision |
|-------------|------------|-----------|-----------|-----------------|------------------------|----------------|--------|----------|
| shared-legacy-app | shared-legacy-app | [BookCatalog.Web](repos/shared-legacy-app/report.md) | .NET Framework 4.8 | App Service MI | Migrate to .NET 10 | 1/2/3 | M | Blocked |

## Cost Estimate

**Portfolio monthly estimate: ~$63/month**

### shared-legacy-app (~$63/month)

| Service | SKU | Monthly Cost | Pricing Model |
|---------|-----|-------------|---------------|
| Azure Blob Storage | Hot LRS | ~$5 | Per GB/month |
| Azure Key Vault with Managed Identity | Standard | ~$5 | Per transaction |
| Azure SQL Database | S0 (10 DTU) | $15 | DTU-based |
| Microsoft Entra ID | P1 | ~$6 | Per user/month |
| App Service MI | B1 | $32 | Per instance/hour |
| **Total** | | **~$63** | |

> ⚠️ These cost estimates are directional and based on Azure retail pricing using pre-selected SKUs aligned to a baseline dev/test environment. Consumption-based services use assumed default usage volumes. Actual production costs will vary and are likely to change based on application-specific sizing, performance requirements, real-world usage patterns, selected region, reserved instance or savings plan commitments, and enterprise agreement discounts.

---

[Share feedback](https://aka.ms/ghcp-appmod/feedback)
