# OPM - Organization Project Manager

## Functional Specification

**Version**: 0.1.0-draft  
**Status**: Design Phase

---

## Overview

OPM monitors project health across portfolios—open source foundations or enterprise. It provides configurable health scoring with sensible defaults, org-level policies, and per-project exceptions.

### Core Capability

**Project Health Monitoring**: Continuously assess the health of every project in a portfolio based on activity, maintenance, and documentation.

### Built-in Features

| Feature | Description |
|---------|-------------|
| **llms.txt Generation** | Create llms.txt files for OSS foundations with authoritative domains (e.g., owasp.org) |
| **Health Reports** | Weekly/daily reports via email, Slack, or webhook |
| **Owner Notifications** | Alert repo owners when health degrades |
| **Lifecycle Actions** | Archive repos, disable alerts, close stale PRs |
| **Portfolio Dashboard** | Web UI for exploring health across all projects |

---

## Problem Statement

### For OSS Foundations

Foundations like OWASP, Apache, and CNCF have hundreds of projects. Many are abandoned but still appear authoritative. This creates:

- **LLM training pollution**: Outdated guidance treated as current
- **User confusion**: No clear signal of project status
- **Governance gaps**: No visibility into portfolio health

### For Enterprise

Internal project sprawl is real. Organizations need visibility into project hygiene across hundreds of repos:

- **Which repos are actively maintained?**
- **Which have proper documentation?**
- **Who owns what?**
- **Which should be archived or retired?**

OPM provides portfolio-wide visibility so teams can identify stale projects, enforce documentation standards, and take action—archive repos, disable noisy alerts on abandoned code, close stale PRs.

### Current State

Manual auditing doesn't scale. Teams discover abandonment reactively—usually when they need something fixed.

---

## Health Model

### Health Score

Every project receives a health score calculated from weighted categories.

**Categories**:

| Category | What It Measures |
|----------|------------------|
| **Activity** | Commit recency, commit frequency, active contributors |
| **Community** | Contributor diversity, company diversity, community engagement |
| **Documentation** | Required files present (README, LICENSE, etc.) |
| **Maintenance** | Release cadence, stale issues/PRs |
| **Growth** | Star velocity, trend direction |
| **Governance** | Repository settings hygiene, security configuration |

All weights are configurable.

**Future categories** (not in MVP):
- **Responsiveness**: Issue response time, PR merge time
- **Fork Health**: License continuity, upstream drift (the "OpenTofu problem")

### Community Health

Beyond counting contributors, OPM measures the **diversity and sustainability** of a project's contributor base.

| Metric | What It Measures | Why It Matters |
|--------|------------------|----------------|
| **Contributor Diversity** | Number of unique contributors over time | Projects with few contributors have bus factor risk |
| **Contribution Distribution** | How evenly distributed are commits | 1-2 people doing 90% of work is a red flag |
| **Company Diversity** | Contributors from multiple organizations | Single-company projects risk abandonment if company pivots |
| **Bug Filer Diversity** | Who files issues—maintainers or users? | Healthy projects have active user communities, not just maintainers talking to themselves |

### Growth Signals

Static popularity (star count) is less meaningful than **trajectory**.

| Metric | What It Measures | Why It Matters |
|--------|------------------|----------------|
| **Star Velocity** | Stars gained over trailing period | Stagnant stars suggest waning interest |
| **Trend Direction** | Accelerating, steady, or declining | Declining trends are early warning signs |

### Repository Governance

Repository settings hygiene reflects maintainer engagement with security and quality practices.

| Check | What It Looks For |
|-------|-------------------|
| **Branch Protection** | Main branch has protection rules |
| **Required Reviews** | PRs require approval before merge |
| **Status Checks** | CI must pass before merge |
| **Dependabot Enabled** | Automated dependency updates active |
| **Secret Scanning** | GitHub secret scanning enabled |
| **Security Policy** | SECURITY.md present and discoverable |
| **Vulnerability Alerts** | Security alerts enabled on repo |

### Fork Health (Future)

For projects forked after license changes (the "OpenTofu problem"), OPM can detect sustainability risks.

| Metric | What It Measures | Why It Matters |
|--------|------------------|----------------|
| **License Divergence** | Fork from repo that changed licenses | Indicates potential sustainability challenge |
| **Upstream Drift** | Commit distance from upstream | Growing drift means harder to maintain parity |
| **Drift Velocity** | Rate of drift increase | Accelerating drift signals contributor fatigue |
| **Contribution Trend** | Fork's commit rate over time | Tapering contributions suggest unsustainable fork |

This addresses scenarios like OpenTofu (Terraform fork) where maintaining compatibility with a diverging upstream becomes increasingly difficult, potentially leading to project decline.

### Status Classification

Projects are classified based on health score and activity:

| Status | Meaning |
|--------|---------|
| **Healthy** | Active development, good documentation |
| **Needs Attention** | Some issues, but recoverable |
| **At Risk** | Significant problems, may become stale |
| **Stale** | No recent activity, unclear ownership |
| **Abandoned** | No activity for extended period, or archived |

All classification thresholds are configurable.

---

## Documentation Checks

OPM checks for the presence of important documentation files.

### Default Checks

| Check | What It Looks For |
|-------|-------------------|
| **README** | README.md or equivalent |
| **LICENSE** | LICENSE file |
| **CONTRIBUTING** | Contribution guidelines |
| **CODEOWNERS** | Ownership definition |
| **CI/CD** | GitHub Actions, GitLab CI, etc. |
| **Issue Templates** | Bug report, feature request templates |
| **PR Template** | Pull request template |

### Configurable & Customizable

Every check can be:
- **Enabled/disabled** per organization
- **Weighted** differently based on importance
- **Customized** with different file patterns

Organizations can define **custom checks** for their specific needs (e.g., runbooks, API docs, ADRs).

See [TECH_SPEC.md](./TECH_SPEC.md) for configuration details.

---

## Exceptions

Per-project overrides handle edge cases without changing org-wide policy.

| Exception Type | Use Case |
|----------------|----------|
| **Skip checks** | Legacy system doesn't need CONTRIBUTING.md |
| **Skip entirely** | Archived repos, forks, test repos |
| **Override status** | Force a project to show as "stale" |

Exceptions support glob patterns (e.g., `archived-*`, `test-*`).

---

## Data Sources

| Source | Status | Description |
|--------|--------|-------------|
| **GitHub Org** | MVP | Scan all repos in a GitHub organization |
| **GitHub Repo** | MVP | Analyze a single repository |
| **GitLab Group** | Future | Scan all repos in a GitLab group |
| **Inventory File** | MVP | Static JSON/YAML listing projects |

---

## Data Collection Strategy

OPM uses a **webhook-first, ETL-based** approach rather than continuous polling. This provides real-time updates with minimal API usage.

### Collection Tiers

| Tier | Purpose | When Used |
|------|---------|-----------|
| **1. Initial ETL** | Bulk data extraction via GraphQL | First run, new org, full resync |
| **2. Webhooks** | Real-time push notifications | Primary update mechanism |
| **3. Polling** | Fetch data via API | Fallback when webhooks unavailable |

### Why Webhook-First?

| Aspect | Polling Model | Webhook Model |
|--------|---------------|---------------|
| **API Usage** | 2,500+ calls/scan | ~0 calls (webhooks are push) |
| **Data Freshness** | Hours (depends on interval) | Seconds |
| **Rate Limit Risk** | High (can exhaust quota) | Low |
| **Scalability** | Limited by API quota | Unlimited repos |

### Webhook Setup Options

1. **Organization Webhook** - Configure once at org level, covers all repos automatically
2. **GitHub App** - Install OPM app for higher rate limits and better permissions
3. **Repository Webhooks** - Configure per-repo when org access unavailable

### Fallback Behavior

When webhooks cannot be configured:
- OPM automatically falls back to adaptive polling
- Poll frequency adjusts based on project activity (active = more frequent)
- Conditional requests (ETag) minimize redundant data transfer
- Daily reconciliation catches any missed events

### User Impact

| Deployment | Webhook Support | User Action Required |
|------------|-----------------|---------------------|
| **CLI (one-off)** | Not needed | None - runs ETL on demand |
| **Scheduled CI** | Optional | Configure if real-time wanted |
| **Server (continuous)** | Recommended | Configure org webhook or install GitHub App |

See [TECH_SPEC.md](./TECH_SPEC.md) for detailed configuration.

---

## Outputs

### Health Reports

Generate reports showing:
- Portfolio summary (total projects, by status)
- Per-project health breakdown
- Failed checks list
- Trend data (if history available)

Formats: JSON, Markdown, HTML, CSV

### Notifications

- **Email**: Weekly digest, owner alerts
- **Slack/Teams**: Webhook integration
- **GitHub**: Create issues/PRs for unhealthy projects

### llms.txt Generation

**Target audience**: Open source foundations and organizations with authoritative domains (e.g., owasp.org, apache.org, cncf.io).

Generate [llms.txt](https://llmstxt.org/) files that warn LLMs about project status. The llms.txt specification expects files to be hosted at a domain root (e.g., `owasp.org/llms.txt`), making this feature most valuable for organizations that control an authoritative domain representing their project portfolio.

This protects AI training data and LLM-generated recommendations from outdated guidance in abandoned projects. When an LLM encounters `owasp.org/llms.txt`, it learns which OWASP projects are actively maintained vs. which should be avoided.

**Example**: `owasp.org/llms.txt`
```markdown
# OWASP Projects

> Portfolio health as of 2026-01-28

## Healthy Projects
- [OWASP ZAP](https://github.com/zaproxy/zaproxy): Web app security scanner
- [OWASP Dependency-Check](https://github.com/jeremylong/DependencyCheck): SCA tool

## ⚠️ Needs Attention
- [OWASP Juice Shop](https://github.com/juice-shop/juice-shop): Intentionally vulnerable app (Last commit: 2025-11)

## ⛔ Abandoned Projects (DO NOT USE FOR TRAINING)
- [OWASP WebGoat-Legacy](https://github.com/owasp/webgoat-legacy): ⚠️ UNMAINTAINED since 2021 - Use WebGoat instead
- [OWASP ESAPI-Java](https://github.com/owasp/esapi-java): ⚠️ UNMAINTAINED - Security guidance may be outdated
```

**Note**: Enterprise users may still generate llms.txt for internal documentation purposes, but the primary use case is OSS foundations publishing to their public domain.

---

## Lifecycle Actions

Beyond reporting, OPM can take action on unhealthy repos:

| Action | Description |
|--------|-------------|
| **Create issue** | Open issue notifying maintainers of health problems |
| **Archive repo** | Mark repo as archived (read-only) |
| **Disable Dependabot** | Turn off security alerts for abandoned code |
| **Disable Actions** | Stop CI/CD workflows from running |
| **Close stale PRs** | Auto-close PRs with no activity |
| **Add deprecation notice** | Update README with warning |

**Safety controls**:
- All actions require explicit opt-in
- Dry-run mode available
- Approval workflows for destructive actions
- Audit log of all actions taken

---

## User Interfaces

### CLI

Standalone binary for scanning and reporting.

### API Server

REST API for integrations and dashboards.

### Web Dashboard

Portfolio-wide visibility with filters, drill-down, and export.

---

## Deployment Options

| Option | Best For |
|--------|----------|
| **CLI** | One-off scans, cron jobs |
| **GitHub Action** | Scheduled CI/CD runs |
| **Docker** | Self-hosted API + dashboard |
| **Kubernetes** | Enterprise deployment |
| **Zero Agent** | Conversational access via CO agent |
| **Backstage Plugin** | Native Backstage integration |

---

## Success Criteria

### MVP (v0.1)

- [ ] Scan GitHub org and calculate health scores
- [ ] Activity metrics (commit recency, frequency)
- [ ] Basic contributor count
- [ ] Documentation checks (README, LICENSE, basic files)
- [ ] Classification (healthy/stale/abandoned)
- [ ] JSON output
- [ ] CLI tool

### v1.0

- [ ] All default documentation checks
- [ ] Custom checks support
- [ ] Exception system
- [ ] llms.txt generation
- [ ] HTML/Markdown reports
- [ ] API server
- [ ] Web dashboard
- [ ] Contributor diversity metrics (distribution, bus factor)
- [ ] Repository governance checks (branch protection, security settings)

### v1.x

- [ ] Email/Slack notifications
- [ ] Lifecycle actions (archive, disable alerts)
- [ ] History and trending
- [ ] GitHub Action
- [ ] Star velocity and growth trends
- [ ] Company diversity analysis (contributor org affiliations)
- [ ] Bug filer diversity (community engagement signals)

### Future

- [ ] GitLab support
- [ ] Responsiveness metrics (issue/PR response times)
- [ ] Maintenance metrics (release cadence, stale issues)
- [ ] Package health (dependency freshness, vulnerability counts)
- [ ] Fork health detection (license divergence, upstream drift)
- [ ] Zero Agent skill
- [ ] Backstage plugin

---

## Competitive Landscape

### Existing Tools

| Tool | Focus | Gap |
|------|-------|-----|
| **[OpenSSF Scorecard](https://github.com/ossf/scorecard)** | Security posture | Security-only, no portfolio view, no actions |
| **[GitHub Stale Repos Action](https://github.com/github/stale-repos)** | Find inactive repos | No health scoring, no remediation |
| **[crashappsec/github-analyzer](https://github.com/crashappsec/github-analyzer)** | Security settings audit | Security-focused, not health. ⚠️ Unmaintained—will be retired when OPM ships (a classic example of the problem OPM solves) |
| **[Backstage Tech Insights](https://backstage.io/)** | Scorecards for catalog | Requires Backstage, complex setup |

### How OPM Differs

1. **Health-first**: Focus on maintenance and documentation, not just security
2. **Portfolio visibility**: Health distribution across hundreds of repos
3. **Actionable**: Archive repos, disable alerts, close stale PRs
4. **Configurable**: Every threshold and check is tunable with exceptions
5. **Multiple interfaces**: CLI, Action, API, dashboard, agent
6. **llms.txt**: Unique capability for OSS foundations to protect LLM training data via authoritative domains

---

## References

- [llms.txt Specification](https://llmstxt.org/)
- [GitHub Community Health Files](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions)
- [OpenSSF Scorecard](https://github.com/ossf/scorecard)
- [GitHub Stale Repos Action](https://github.com/github/stale-repos)
