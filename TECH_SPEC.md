# OPM - Technical Specification

**Version**: 0.1.0-draft  
**Status**: Design Phase

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                           OPM System                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │   opm CLI    │  │  opm server  │  │    opm ui    │              │
│  │    (Go)      │  │  (Go + API)  │  │ (Node + TW)  │              │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘              │
│         │                 │                  │                       │
│         └─────────────────┼──────────────────┘                       │
│                           ▼                                          │
│  ┌─────────────────────────────────────────────────────────┐        │
│  │                 Core Library (Go)                        │        │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐       │        │
│  │  │ Scanner │ │ Scoring │ │ Checks  │ │ Config  │       │        │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘       │        │
│  └─────────────────────────────────────────────────────────┘        │
│                           │                                          │
│         ┌─────────────────┼─────────────────┐                       │
│         ▼                 ▼                 ▼                       │
│  ┌───────────┐     ┌───────────┐     ┌───────────┐                 │
│  │  GitHub   │     │  GitLab   │     │ Inventory │                 │
│  └───────────┘     └───────────┘     └───────────┘                 │
│                                                                      │
├─────────────────────────────────────────────────────────────────────┤
│                      Outputs (Features)                              │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐               │
│  │ llms.txt │ │ Reports  │ │ Actions  │ │  Notify  │               │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘               │
└─────────────────────────────────────────────────────────────────────┘
```

### Components

| Component | Language | Description |
|-----------|----------|-------------|
| `opm` CLI | Go | Standalone scanner, runs anywhere |
| `opm server` | Go | REST API for integrations |
| `opm ui` | Node + Tailwind | Web dashboard |
| Core Library | Go | Shared scanning, scoring, config logic |

---

## Data Collection Architecture

OPM uses a **three-tier data collection strategy** that prioritizes real-time updates over polling:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        DATA COLLECTION LAYER                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  TIER 1: INITIAL ETL (Bulk Scrape)                                          │
│  ─────────────────────────────────                                          │
│  - GraphQL batch queries (100 repos/request)                                │
│  - Resumable with checkpoints                                               │
│  - Used for: first run, new org added, full resync                         │
│                                                                              │
│  TIER 2: WEBHOOKS (Primary - Real-time)                                     │
│  ──────────────────────────────────────                                     │
│  - GitHub org/repo webhooks or GitHub App                                   │
│  - GitLab group/project webhooks                                            │
│  - Instant updates on push, release, issues, PRs                           │
│  - Zero API calls for covered repos                                         │
│                                                                              │
│  TIER 3: POLLING (Fallback Only)                                            │
│  ────────────────────────────────                                           │
│  - Only for repos without webhook coverage                                  │
│  - Adaptive intervals based on activity level                              │
│  - Conditional requests (ETag/If-Modified-Since)                           │
│  - Daily reconciliation to catch missed events                             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
                    ┌───────────────────────────┐
                    │     Event Processor       │
                    │  (Normalizes all inputs)  │
                    └───────────────────────────┘
                                    │
                                    ▼
                    ┌───────────────────────────┐
                    │      Data Store           │
                    │   (SQLite/PostgreSQL)     │
                    └───────────────────────────┘
```

### Collection Mode Priority

| Mode | When Used | API Cost | Latency |
|------|-----------|----------|---------|
| **Webhooks** | Always preferred | Zero | Seconds |
| **ETL** | Initial load, full resync | Low (GraphQL batched) | Minutes |
| **Polling** | Fallback when no webhook | Higher | Hours |

### Webhook Event Mapping

#### GitHub Events → OPM Data

| Webhook Event | Data Updated | Health Impact |
|---------------|--------------|---------------|
| `push` | commits, pushedAt, contributors | Activity score |
| `release` | releases, lastRelease | Maintenance score |
| `issues` | openIssues, closedIssues | Maintenance score |
| `pull_request` | openPRs, mergedPRs | Maintenance score |
| `star` | starCount | Growth score |
| `repository` | archived, visibility | Status classification |
| `branch_protection_rule` | branchProtection | Governance score |
| `member` | collaborators | Contributor count |

#### GitLab Events → OPM Data

| Webhook Event | Data Updated | Health Impact |
|---------------|--------------|---------------|
| `push` | commits, pushedAt | Activity score |
| `tag_push` | tags, releases | Maintenance score |
| `issue` | openIssues | Maintenance score |
| `merge_request` | openMRs, mergedMRs | Maintenance score |
| `release` | releases | Maintenance score |
| `pipeline` | CI status | Governance score |

### Webhook Configuration Options

1. **Organization Webhook** (Recommended for enterprises)
   - Single webhook covers all repos
   - Requires org admin access
   - Auto-covers new repos

2. **GitHub App** (Recommended for OSS foundations)
   - Higher rate limits
   - Granular permissions
   - Multi-org support

3. **Repository Webhooks** (Fallback)
   - When org-level access unavailable
   - Manual setup per repo

### Adaptive Polling Intervals

When webhooks unavailable, polling adapts to activity level:

| Project Status | Poll Interval | Rationale |
|----------------|---------------|-----------|
| Healthy | 6 hours | Active, changes often |
| Needs Attention | 12 hours | Some activity |
| At Risk | 24 hours | Infrequent changes |
| Stale | 7 days | Unlikely to change |
| Abandoned | 30 days | Verify still archived |

---

## Project Structure

```
OPM/
├── CLAUDE.md                    # Agent instructions
├── README.md                    # Project overview
├── FUNCTIONAL_SPEC.md           # What OPM does
├── TECH_SPEC.md                 # How it's built (this file)
├── Makefile                     
├── go.mod / go.sum              
│
├── cmd/opm/                     # CLI entry points
│   ├── main.go
│   ├── scan.go
│   ├── check.go
│   ├── generate.go
│   └── server.go
│
├── internal/
│   ├── config/                  # Configuration
│   ├── collector/               # Data collection orchestration
│   │   ├── collector.go         # Main collector interface
│   │   ├── etl.go               # Initial bulk extraction (GraphQL)
│   │   ├── webhook.go           # Webhook event processor
│   │   ├── poller.go            # Fallback polling logic
│   │   └── coverage.go          # Track webhook vs polling coverage
│   ├── webhook/                 # Webhook handling
│   │   ├── handler.go           # HTTP handlers for webhook endpoints
│   │   ├── github.go            # GitHub webhook parsing/validation
│   │   ├── gitlab.go            # GitLab webhook parsing/validation
│   │   └── signature.go         # HMAC signature verification
│   ├── scanner/                 # API clients for data fetching
│   │   ├── github.go            # GitHub API client (GraphQL + REST)
│   │   ├── gitlab.go            # GitLab API client
│   │   ├── inventory.go         # Static inventory loader
│   │   └── ratelimit.go         # Rate limit tracking & conditional requests
│   ├── health/                  # Score calculation
│   ├── checks/                  # Documentation checks
│   ├── exceptions/              # Per-project overrides
│   ├── outputs/                 # llms.txt, reports
│   ├── actions/                 # Archive, disable, close
│   ├── notify/                  # Email, Slack, Teams
│   ├── api/                     # REST API
│   └── store/                   # SQLite, memory, sync tracking
│
├── pkg/models/                  # Public types
│
├── ui/                          # Web UI (Node project)
│
├── configs/                     # Example configs
│
└── test/                        # Tests and fixtures
```

---

## Configuration

OPM uses a single YAML configuration file. Everything is optional—OPM works with sensible defaults.

### Minimal Config

```yaml
# opm.yaml - just specify what to scan
sources:
  - type: github_org
    org: your-company
```

### Full Configuration Reference

```yaml
# opm.yaml - complete configuration

organization:
  name: "ACME Corp"
  slug: "acme"

# =============================================================================
# SOURCES - Where to find projects
# =============================================================================
sources:
  - type: github_org
    org: "acme-corp"
    token: "${GITHUB_TOKEN}"        # env var substitution
    include_archived: false
    exclude:
      - "*-deprecated"
      - "test-*"
      - "fork-*"
  
  - type: gitlab_group
    group: "acme/platform"
    url: "https://gitlab.acme.com"
    token: "${GITLAB_TOKEN}"
  
  - type: inventory
    url: "https://example.com/projects.json"
    mapping:
      name: "title"
      repo_url: "repository"

# =============================================================================
# DATA COLLECTION - How OPM gets and updates data
# =============================================================================
collection:
  # Collection mode: "webhook" (preferred), "polling", or "hybrid" (default)
  mode: hybrid

  # Initial ETL settings (bulk data extraction)
  etl:
    concurrency: 4              # Parallel GraphQL requests
    batch_size: 100             # Repos per GraphQL query
    resumable: true             # Save progress for interrupted ETL

  # Webhook settings (real-time updates)
  webhooks:
    enabled: true

    # Secrets for signature validation
    github_secret: "${GITHUB_WEBHOOK_SECRET}"
    gitlab_secret: "${GITLAB_WEBHOOK_SECRET}"

    # GitHub App (alternative to org webhooks, higher rate limits)
    github_app:
      enabled: false
      app_id: "${GITHUB_APP_ID}"
      private_key_path: "/etc/opm/github-app.pem"
      webhook_secret: "${GITHUB_APP_WEBHOOK_SECRET}"

    # Events to process (others ignored)
    github_events:
      - push
      - release
      - issues
      - pull_request
      - repository
      - star
      - member
      - branch_protection_rule

    gitlab_events:
      - push
      - tag_push
      - issue
      - merge_request
      - release
      - pipeline

  # Polling settings (fallback when webhooks unavailable)
  polling:
    fallback_only: true         # Only poll repos without webhook coverage

    # Adaptive intervals based on project health status
    intervals:
      healthy: "6h"
      needs_attention: "12h"
      at_risk: "24h"
      stale: "168h"             # 7 days
      abandoned: "720h"         # 30 days

    # Use conditional requests (ETag/If-Modified-Since) to save rate limit
    conditional_requests: true

    # Reserve this % of rate limit for on-demand requests
    rate_limit_reserve: 20

    # Multiple tokens for higher throughput (optional)
    token_rotation:
      enabled: false
      tokens:
        - "${GITHUB_TOKEN_1}"
        - "${GITHUB_TOKEN_2}"

  # Data reconciliation (catch missed webhook events)
  reconciliation:
    daily_sync: true            # Quick sync to catch missed events
    daily_sync_time: "03:00"    # 3 AM
    weekly_full_sync: true      # Full data verification
    weekly_full_sync_day: "sunday"
    weekly_full_sync_time: "02:00"

# =============================================================================
# HEALTH SCORING - Weights and thresholds
# =============================================================================
health:
  # Category weights (must sum to 100)
  weights:
    activity: 40
    documentation: 40
    maintenance: 20
  
  # Activity scoring thresholds
  activity:
    commit_recency:
      weight: 50              # Weight within activity category
      thresholds:
        - max_days: 30
          score: 100
        - max_days: 90
          score: 75
        - max_days: 180
          score: 50
        - max_days: 365
          score: 25
        - max_days: 9999
          score: 0
    
    commit_frequency:
      weight: 30
      thresholds:
        - min_per_month: 10
          score: 100
        - min_per_month: 4
          score: 75
        - min_per_month: 1
          score: 50
        - min_per_month: 0
          score: 25
    
    contributors:
      weight: 20
      thresholds:
        - min_active_6mo: 3
          score: 100
        - min_active_6mo: 2
          score: 75
        - min_active_6mo: 1
          score: 50
  
  # Maintenance scoring thresholds (MVP: simplified)
  maintenance:
    has_releases:
      weight: 50
      # true = 100, false = 0

    stale_issues:
      weight: 25
      thresholds:
        - max_count: 5
          score: 100
        - max_count: 10
          score: 75
        - max_count: 20
          score: 50
        - max_count: 9999
          score: 25

    stale_prs:
      weight: 25
      thresholds:
        - max_count: 2
          score: 100
        - max_count: 5
          score: 75
        - max_count: 10
          score: 50

  # =============================================================================
  # COMMUNITY SCORING (v1.0+) - Contributor diversity and sustainability
  # =============================================================================
  community:
    enabled: true
    weight: 15                    # Weight in overall health (adjust others to sum to 100)

    # Bus factor - minimum contributors responsible for 50% of work
    bus_factor:
      weight: 40
      thresholds:
        - min_bus_factor: 5
          score: 100
        - min_bus_factor: 3
          score: 75
        - min_bus_factor: 2
          score: 50
        - min_bus_factor: 1
          score: 25

    # Contribution concentration - top contributor's share
    concentration:
      weight: 30
      thresholds:
        - max_top_percent: 40     # Top contributor < 40% = healthy
          score: 100
        - max_top_percent: 60
          score: 75
        - max_top_percent: 80
          score: 50
        - max_top_percent: 100
          score: 25

    # Company diversity (requires profile lookups, may hit rate limits)
    company_diversity:
      enabled: false              # Disabled by default due to API cost
      weight: 15
      thresholds:
        - min_companies: 3
          score: 100
        - min_companies: 2
          score: 50
        - min_companies: 1
          score: 25

    # Community engagement - external bug filers vs maintainers
    community_engagement:
      weight: 15
      thresholds:
        - min_external_filer_ratio: 0.5    # 50%+ issues from non-maintainers
          score: 100
        - min_external_filer_ratio: 0.25
          score: 75
        - min_external_filer_ratio: 0.1
          score: 50
        - min_external_filer_ratio: 0
          score: 25

  # =============================================================================
  # GROWTH SCORING (v1.x) - Star velocity and trends
  # =============================================================================
  growth:
    enabled: true
    weight: 10                    # Weight in overall health

    star_velocity:
      weight: 60
      thresholds:
        - min_stars_per_month: 10
          score: 100
        - min_stars_per_month: 5
          score: 75
        - min_stars_per_month: 1
          score: 50
        - min_stars_per_month: 0
          score: 25              # Stagnant

    star_trend:
      weight: 40
      # "accelerating" = 100, "steady" = 75, "declining" = 50, "stagnant" = 25

  # =============================================================================
  # GOVERNANCE SCORING (v1.0+) - Repository settings hygiene
  # =============================================================================
  governance:
    enabled: true
    weight: 10                    # Weight in overall health

    # Which settings to check and their weights
    checks:
      branch_protection:
        enabled: true
        weight: 25
      require_reviews:
        enabled: true
        weight: 20
      require_status_checks:
        enabled: true
        weight: 15
      dependabot_enabled:
        enabled: true
        weight: 15
      secret_scanning:
        enabled: true
        weight: 10
      vulnerability_alerts:
        enabled: true
        weight: 10
      signed_commits:
        enabled: false           # Often not practical
        weight: 5

  # =============================================================================
  # FORK HEALTH (Future) - License divergence and upstream drift
  # =============================================================================
  fork_health:
    enabled: false               # Disabled by default, opt-in
    weight: 0                    # Add weight when enabled

    # Alert if fork is from a repo that changed licenses
    license_divergence:
      enabled: true
      severity: high             # How serious is license divergence?

    # Alert if drift from upstream is accelerating
    upstream_drift:
      enabled: true
      thresholds:
        - max_commits_behind: 100
          score: 100
        - max_commits_behind: 500
          score: 75
        - max_commits_behind: 1000
          score: 50
        - max_commits_behind: 9999
          score: 25

    # Track contribution trends on fork
    fork_sustainability:
      enabled: true
      # Score based on fork_commit_trend: increasing=100, steady=75, declining=50

# =============================================================================
# CLASSIFICATION - Status thresholds
# =============================================================================
classification:
  # All thresholds are configurable
  healthy:
    min_score: 70
    max_days_since_commit: 90
  
  needs_attention:
    min_score: 50
    max_days_since_commit: 180
  
  at_risk:
    min_score: 30
    max_days_since_commit: 270
  
  stale:
    min_score: 0
    max_days_since_commit: 365
  
  # Below stale thresholds OR archived = abandoned

# =============================================================================
# DOCUMENTATION CHECKS - Fully configurable
# =============================================================================
checks:
  # Each check can be enabled/disabled, weighted, customized
  - id: readme
    name: "README"
    enabled: true
    severity: critical          # critical, high, medium, low, info
    weight: 30                  # Weight within documentation score
    files:                      # Glob patterns
      - "README.md"
      - "README.rst"
      - "README.txt"
      - "README"
    content:                    # Optional: require certain content
      - pattern: "install|setup|getting.started"
        flags: "i"
        description: "Installation instructions"
        required: false         # Warn but don't fail
  
  - id: license
    name: "License"
    enabled: true
    severity: critical
    weight: 25
    files:
      - "LICENSE"
      - "LICENSE.md"
      - "LICENSE.txt"
      - "COPYING"
    allowed_licenses:           # Optional: restrict to approved licenses
      - "MIT"
      - "Apache-2.0"
      - "BSD-3-Clause"
  
  - id: contributing
    name: "Contributing Guide"
    enabled: true
    severity: high
    weight: 15
    files:
      - "CONTRIBUTING.md"
      - ".github/CONTRIBUTING.md"
  
  - id: codeowners
    name: "Code Owners"
    enabled: true
    severity: medium
    weight: 15
    files:
      - "CODEOWNERS"
      - ".github/CODEOWNERS"
  
  - id: ci_config
    name: "CI/CD Configuration"
    enabled: true
    severity: medium
    weight: 10
    files:
      - ".github/workflows/*.yml"
      - ".gitlab-ci.yml"
      - "Jenkinsfile"
      - ".circleci/config.yml"
  
  - id: security_policy
    name: "Security Policy"
    enabled: false              # Disabled by default
    severity: medium
    weight: 5
    files:
      - "SECURITY.md"
  
  - id: issue_templates
    name: "Issue Templates"
    enabled: true
    severity: low
    weight: 5
    files:
      - ".github/ISSUE_TEMPLATE/*.md"
      - ".github/ISSUE_TEMPLATE/*.yml"
      - ".github/ISSUE_TEMPLATE.md"
      - ".gitlab/issue_templates/*.md"
  
  - id: pr_template
    name: "PR Template"
    enabled: true
    severity: low
    weight: 5
    files:
      - ".github/PULL_REQUEST_TEMPLATE.md"
      - ".github/PULL_REQUEST_TEMPLATE/*.md"
      - ".gitlab/merge_request_templates/*.md"
  
  # Custom checks - define your own
  - id: runbook
    name: "Operational Runbook"
    enabled: false
    severity: high
    weight: 10
    files:
      - "docs/runbook.md"
      - "RUNBOOK.md"
      - "docs/operations.md"
  
  - id: api_docs
    name: "API Documentation"
    enabled: false
    severity: medium
    weight: 10
    files:
      - "docs/api.md"
      - "openapi.yaml"
      - "swagger.yaml"

# =============================================================================
# EXCEPTIONS - Per-project overrides
# =============================================================================
exceptions:
  - repo: "acme-corp/legacy-monolith"
    reason: "Legacy system in maintenance mode"
    skip_checks:
      - contributing
      - ci_config
    override_status: stale      # Force status
    
  - repo: "acme-corp/infrastructure"
    reason: "Internal tooling, different standards"
    override_thresholds:
      activity:
        commit_recency:
          thresholds:
            - max_days: 180     # More lenient
              score: 100
    skip_checks:
      - codeowners
  
  - repo: "acme-corp/archived-*"    # Glob pattern
    reason: "Archived projects"
    skip_entirely: true
  
  - repo: "^acme-corp/test-.*$"     # Regex (starts with ^)
    reason: "Test repositories"
    skip_entirely: true

# =============================================================================
# OUTPUTS
# =============================================================================
outputs:
  # llms.txt is primarily for OSS foundations with authoritative domains
  # (e.g., owasp.org/llms.txt, apache.org/llms.txt)
  llms_txt:
    enabled: true
    path: "./output/llms.txt"
    include_abandoned: true
    abandoned_warning: "⚠️ UNMAINTAINED - Do not use for training"
  
  reports:
    enabled: true
    formats:
      - json
      - markdown
      - html
    path: "./output/reports/"

# =============================================================================
# LIFECYCLE ACTIONS (opt-in)
# =============================================================================
actions:
  create_issue:
    enabled: false
    threshold: at_risk           # Create issues starting at this status
    title_template: "[OPM] Project health degraded: {{.Project.Name}}"
    labels:
      - "opm"
      - "health"

  archive:
    enabled: false
    dry_run: true               # Preview without executing
    require_approval: true
    threshold: abandoned        # Only archived status
  
  disable_dependabot:
    enabled: false
    threshold: abandoned
  
  disable_actions:
    enabled: false
    threshold: abandoned
  
  close_stale_prs:
    enabled: false
    days_inactive: 90
    comment: "Closing due to inactivity. Please reopen if still needed."

# =============================================================================
# NOTIFICATIONS
# =============================================================================
notifications:
  email:
    enabled: false
    provider: smtp              # smtp, sendgrid, ses, resend
    from: "opm@acme.com"
    schedules:
      - cron: "0 9 * * 1"       # Monday 9 AM
        recipients:
          - "engineering-leads@acme.com"
        type: weekly_summary
  
  slack:
    enabled: false
    webhook_url: "${SLACK_WEBHOOK_URL}"
    channel: "#engineering-health"
    triggers:
      - on: status_change
        to: abandoned

# =============================================================================
# SERVER
# =============================================================================
server:
  host: "0.0.0.0"
  port: 8080
  auth:
    enabled: false
    type: api_key
    api_key: "${OPM_API_KEY}"
  
  scheduler:
    enabled: false
    cron: "0 */6 * * *"         # Every 6 hours
```

---

## Default Values

When no config is provided, OPM uses these defaults:

### Health Weights

**MVP defaults** (activity, documentation, maintenance only):

| Category | Default |
|----------|---------|
| Activity | 40% |
| Documentation | 40% |
| Maintenance | 20% |

**v1.0+ defaults** (with community and governance):

| Category | Default |
|----------|---------|
| Activity | 30% |
| Community | 15% |
| Documentation | 30% |
| Maintenance | 15% |
| Governance | 10% |

**v1.x+ defaults** (with growth):

| Category | Default |
|----------|---------|
| Activity | 25% |
| Community | 15% |
| Documentation | 25% |
| Maintenance | 15% |
| Growth | 10% |
| Governance | 10% |

### Classification Thresholds

| Status | Min Score | Max Days Since Commit |
|--------|-----------|----------------------|
| Healthy | 70 | 90 |
| Needs Attention | 50 | 180 |
| At Risk | 30 | 270 |
| Stale | 0 | 365 |
| Abandoned | - | >365 or archived |

### Default Checks

| Check | Enabled | Severity | Weight |
|-------|---------|----------|--------|
| README | Yes | Critical | 25% |
| LICENSE | Yes | Critical | 20% |
| CONTRIBUTING | Yes | High | 15% |
| CODEOWNERS | Yes | Medium | 15% |
| CI/CD Config | Yes | Medium | 10% |
| Issue Templates | Yes | Low | 5% |
| PR Template | Yes | Low | 5% |
| SECURITY.md | No | Medium | 5% |

---

## Core Types

### Project

```go
type Project struct {
    ID          string      `json:"id"`
    Name        string      `json:"name"`
    URL         string      `json:"url"`
    Description string      `json:"description,omitempty"`
    Health      Health      `json:"health"`
    Checks      []CheckResult `json:"checks"`
    Metrics     Metrics     `json:"metrics"`
    Exceptions  []string    `json:"exceptions,omitempty"`
    ScannedAt   time.Time   `json:"scanned_at"`
}
```

### Health

```go
type Health struct {
    Score      int         `json:"score"`      // 0-100
    Status     Status      `json:"status"`     // healthy, needs_attention, at_risk, stale, abandoned
    Breakdown  Breakdown   `json:"breakdown"`
}

type Breakdown struct {
    Activity      CategoryScore `json:"activity"`
    Community     CategoryScore `json:"community"`
    Documentation CategoryScore `json:"documentation"`
    Maintenance   CategoryScore `json:"maintenance"`
    Growth        CategoryScore `json:"growth,omitempty"`
    Governance    CategoryScore `json:"governance,omitempty"`
}

type CategoryScore struct {
    Score    int `json:"score"`     // 0-100 for category
    Weight   int `json:"weight"`    // Weight in final score
    Weighted int `json:"weighted"`  // score * weight / 100
}
```

### Status

```go
type Status string

const (
    StatusHealthy       Status = "healthy"
    StatusNeedsAttention Status = "needs_attention"
    StatusAtRisk        Status = "at_risk"
    StatusStale         Status = "stale"
    StatusAbandoned     Status = "abandoned"
)
```

### Metrics

```go
type Metrics struct {
    // Activity
    DaysSinceCommit     int   `json:"days_since_commit"`
    CommitsLast30Days   int   `json:"commits_30d"`
    ActiveContributors  int   `json:"active_contributors_6mo"`

    // Community (v1.0+)
    Community           *CommunityMetrics `json:"community,omitempty"`

    // Growth (v1.x+)
    Growth              *GrowthMetrics    `json:"growth,omitempty"`

    // Governance (v1.0+)
    Governance          *GovernanceMetrics `json:"governance,omitempty"`

    // Fork Health (Future)
    ForkHealth          *ForkHealthMetrics `json:"fork_health,omitempty"`

    // Maintenance
    HasReleases         bool  `json:"has_releases"`
    DaysSinceRelease    int   `json:"days_since_release,omitempty"`
    OpenIssues          int   `json:"open_issues"`
    StaleIssues         int   `json:"stale_issues"`
    OpenPRs             int   `json:"open_prs"`
    StalePRs            int   `json:"stale_prs"`

    // Meta
    Stars               int   `json:"stars"`
    IsArchived          bool  `json:"is_archived"`
    License             string `json:"license,omitempty"`
}

// CommunityMetrics measures contributor diversity and sustainability
type CommunityMetrics struct {
    // Contributor diversity
    TotalContributors      int     `json:"total_contributors"`       // All-time unique contributors
    ContributorsLast6Mo    int     `json:"contributors_6mo"`         // Active in last 6 months
    TopContributorPercent  float64 `json:"top_contributor_percent"`  // % of commits by top contributor
    Top3ContributorPercent float64 `json:"top3_contributor_percent"` // % of commits by top 3
    BusFactor              int     `json:"bus_factor"`               // Minimum contributors for 50% of work

    // Company diversity (requires user profile lookups)
    UniqueCompanies        int      `json:"unique_companies,omitempty"`    // Distinct orgs/companies
    TopCompanyPercent      float64  `json:"top_company_percent,omitempty"` // % from single company
    Companies              []string `json:"companies,omitempty"`           // List of contributing companies

    // Community engagement
    UniqueIssueFilers      int     `json:"unique_issue_filers"`      // Distinct people filing issues
    IssueFilersVsMaintainers float64 `json:"filers_vs_maintainers"`  // Ratio of external filers
    UniqueCommenters       int     `json:"unique_commenters"`        // People engaging in discussions
}

// GrowthMetrics measures popularity trajectory
type GrowthMetrics struct {
    StarsTotal         int     `json:"stars_total"`
    StarsLast30Days    int     `json:"stars_30d"`
    StarsLast90Days    int     `json:"stars_90d"`
    StarVelocity       float64 `json:"star_velocity"`       // Stars per month (trailing avg)
    StarTrend          string  `json:"star_trend"`          // "accelerating", "steady", "declining", "stagnant"
    StarTrendPercent   float64 `json:"star_trend_percent"`  // % change in velocity
    ForksLast30Days    int     `json:"forks_30d,omitempty"`
}

// GovernanceMetrics measures repository settings hygiene
type GovernanceMetrics struct {
    BranchProtection     bool `json:"branch_protection"`      // Main branch protected
    RequireReviews       bool `json:"require_reviews"`        // PRs require approval
    RequireStatusChecks  bool `json:"require_status_checks"`  // CI must pass
    DependabotEnabled    bool `json:"dependabot_enabled"`     // Automated dep updates
    SecretScanning       bool `json:"secret_scanning"`        // Secret scanning enabled
    VulnerabilityAlerts  bool `json:"vulnerability_alerts"`   // Security alerts on
    SignedCommits        bool `json:"signed_commits"`         // Require signed commits
    DeleteBranchOnMerge  bool `json:"delete_branch_on_merge"` // Clean up after merge

    // Calculated score (how many are enabled out of relevant checks)
    EnabledCount         int  `json:"enabled_count"`
    TotalChecks          int  `json:"total_checks"`
}

// ForkHealthMetrics detects the "OpenTofu problem" - forks struggling after upstream license changes
type ForkHealthMetrics struct {
    IsFork               bool   `json:"is_fork"`
    UpstreamRepo         string `json:"upstream_repo,omitempty"`         // e.g., "hashicorp/terraform"
    UpstreamLicense      string `json:"upstream_license,omitempty"`      // Current upstream license
    ForkLicense          string `json:"fork_license,omitempty"`          // Fork's license
    LicenseDiverged      bool   `json:"license_diverged"`                // Licenses differ
    LicenseChangeDate    string `json:"license_change_date,omitempty"`   // When upstream changed

    // Drift metrics
    CommitsBehind        int     `json:"commits_behind,omitempty"`       // Commits fork is behind upstream
    DriftVelocity        float64 `json:"drift_velocity,omitempty"`       // Commits falling behind per month
    LastSyncDate         string  `json:"last_sync_date,omitempty"`       // Last merge from upstream

    // Fork sustainability signals
    ForkCommitsLast30d   int     `json:"fork_commits_30d,omitempty"`     // Original work on fork
    ForkCommitTrend      string  `json:"fork_commit_trend,omitempty"`    // "increasing", "steady", "declining"
    UniqueContributors   int     `json:"unique_contributors,omitempty"`  // People working on fork specifically
}
```

### Check

```go
type Check struct {
    ID              string        `yaml:"id"`
    Name            string        `yaml:"name"`
    Enabled         bool          `yaml:"enabled"`
    Severity        Severity      `yaml:"severity"`
    Weight          int           `yaml:"weight"`
    Files           []string      `yaml:"files"`
    Content         []ContentRule `yaml:"content,omitempty"`
    AllowedLicenses []string      `yaml:"allowed_licenses,omitempty"`
}

type CheckResult struct {
    CheckID   string   `json:"check_id"`
    Name      string   `json:"name"`
    Status    string   `json:"status"`  // pass, fail, skip
    Severity  Severity `json:"severity"`
    Message   string   `json:"message,omitempty"`
}
```

### Data Collection Types

```go
// CollectionMode defines how OPM collects data
type CollectionMode string

const (
    CollectionModeWebhook CollectionMode = "webhook"
    CollectionModePolling CollectionMode = "polling"
    CollectionModeHybrid  CollectionMode = "hybrid"
)

// WebhookCoverage tracks webhook status per source
type WebhookCoverage struct {
    SourceID       string    `json:"source_id"`
    SourceType     string    `json:"source_type"`      // github_org, github_repo, gitlab_group
    WebhookType    string    `json:"webhook_type"`     // org_webhook, app, repo_webhook, none
    Status         string    `json:"status"`           // active, pending, failed, none
    LastEventAt    time.Time `json:"last_event_at"`
    EventsReceived int64     `json:"events_received"`
    LastError      string    `json:"last_error,omitempty"`
}

// SyncState tracks synchronization status
type SyncState struct {
    SourceID            string    `json:"source_id"`
    LastFullSync        time.Time `json:"last_full_sync"`
    LastIncrementalSync time.Time `json:"last_incremental_sync"`
    LastWebhookEvent    time.Time `json:"last_webhook_event"`
    ETLCursor           string    `json:"etl_cursor,omitempty"`  // For resumable ETL
    ETLComplete         bool      `json:"etl_complete"`
}

// WebhookEvent represents a normalized webhook event
type WebhookEvent struct {
    ID          string                 `json:"id"`           // Delivery ID (X-GitHub-Delivery)
    Source      string                 `json:"source"`       // github, gitlab
    EventType   string                 `json:"event_type"`   // push, release, issues, etc.
    RepoID      string                 `json:"repo_id"`
    RepoName    string                 `json:"repo_name"`
    OrgName     string                 `json:"org_name"`
    Payload     map[string]interface{} `json:"payload"`
    ReceivedAt  time.Time              `json:"received_at"`
    ProcessedAt time.Time              `json:"processed_at,omitempty"`
    Error       string                 `json:"error,omitempty"`
}

// CollectionStatus for API response
type CollectionStatus struct {
    Mode                CollectionMode     `json:"mode"`
    Sources             []SourceStatus     `json:"sources"`
    LastETL             time.Time          `json:"last_etl"`
    LastReconciliation  time.Time          `json:"last_reconciliation"`
    RateLimit           RateLimitStatus    `json:"rate_limit"`
}

type SourceStatus struct {
    Type                string    `json:"type"`
    Name                string    `json:"name"`
    WebhookStatus       string    `json:"webhook_status"`
    WebhookType         string    `json:"webhook_type"`
    LastWebhookEvent    time.Time `json:"last_webhook_event"`
    ReposWithCoverage   int       `json:"repos_with_coverage"`
    ReposPollingFallback int      `json:"repos_polling_fallback"`
}

type RateLimitStatus struct {
    Remaining int       `json:"remaining"`
    Limit     int       `json:"limit"`
    ResetAt   time.Time `json:"reset_at"`
}
```

---

## CLI Commands

```bash
# Scanning
opm scan --github-org acme-corp
opm scan --config opm.yaml
opm scan --repo owner/repo

# Quick check
opm check owner/repo
opm check owner/repo --json
opm check owner/repo --explain

# Generate outputs
opm generate llms-txt --config opm.yaml
opm generate report --format html

# Server
opm server --config opm.yaml
opm server --port 8080

# Actions (with --dry-run by default)
opm archive --status abandoned --dry-run
opm disable-dependabot --status abandoned

# Config
opm config validate opm.yaml
opm config defaults
```

---

## API Endpoints

```
# Health & Status
GET  /api/v1/health                    # API health check
GET  /api/v1/projects                  # List all projects
GET  /api/v1/projects/:id              # Project details
GET  /api/v1/projects/:id/history      # Health history

# Data Collection
POST /api/v1/webhooks/github           # GitHub webhook receiver
POST /api/v1/webhooks/gitlab           # GitLab webhook receiver
POST /api/v1/webhooks/github-app       # GitHub App webhook receiver
GET  /api/v1/collection/status         # Collection mode, coverage stats
GET  /api/v1/collection/coverage       # Which repos have webhook coverage
POST /api/v1/collection/sync           # Trigger manual sync
POST /api/v1/collection/etl            # Trigger full ETL

# Reports & Config
POST /api/v1/scan                      # Trigger incremental scan (deprecated, use /collection/sync)
GET  /api/v1/reports/summary           # Portfolio summary
GET  /api/v1/reports/llms-txt          # Generate llms.txt
GET  /api/v1/config                    # Current config
```

---

## Environment Variables

```bash
# Required
GITHUB_TOKEN=ghp_xxxxx

# Optional
GITLAB_TOKEN=glpat-xxxxx
GITHUB_ENTERPRISE_URL=https://github.company.com
GITLAB_URL=https://gitlab.company.com

# Notifications
SLACK_WEBHOOK_URL=https://hooks.slack.com/...
SENDGRID_API_KEY=SG.xxxxx

# Server
OPM_API_KEY=your-api-key
OPM_DB_PATH=./data/opm.db
```

---

## Build & Development

```bash
# Build
make build
make test
make lint

# Run
go run ./cmd/opm scan --github-org test-org
go run ./cmd/opm server --port 8080

# UI
cd ui && npm run dev

# Docker
docker build -t opm .
docker run -e GITHUB_TOKEN=$GITHUB_TOKEN opm scan --github-org your-org
```

---

## Deployment

### Docker Compose

```yaml
services:
  opm-api:
    image: crashappsec/opm:latest
    command: server
    ports:
      - "8080:8080"
    volumes:
      - ./opm.yaml:/etc/opm/config.yaml
    environment:
      - GITHUB_TOKEN=${GITHUB_TOKEN}
  
  opm-ui:
    image: crashappsec/opm-ui:latest
    ports:
      - "3000:3000"
    environment:
      - OPM_API_URL=http://opm-api:8080
```

### GitHub Action

```yaml
name: Project Health
on:
  schedule:
    - cron: '0 9 * * 1'
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: crashappsec/opm-action@v1
        with:
          config: opm.yaml
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

---

## Rate Limiting

| API | Unauthenticated | Authenticated |
|-----|-----------------|---------------|
| GitHub | 60/hour | 5000/hour |
| GitLab | 60/hour | Varies |

Implement:
- Exponential backoff
- Request caching
- Conditional requests (ETag)

---

## Future Features

These features are planned but not in MVP:

### Community Metrics (v1.0)

| Metric | Description | API Source |
|--------|-------------|------------|
| Bus Factor | Min contributors for 50% of commits | `/repos/{owner}/{repo}/stats/contributors` |
| Contribution Distribution | % by top 1, top 3 contributors | Same endpoint, calculate |
| Company Diversity | Distinct orgs in contributor profiles | `/users/{username}` (rate limit heavy) |
| Bug Filer Diversity | External filers vs maintainers | `/repos/{owner}/{repo}/issues` + contributor list |

### Growth Metrics (v1.x)

| Metric | Description | Implementation |
|--------|-------------|----------------|
| Star Velocity | Stars per month average | GitHub Archive or track over time |
| Star Trend | Accelerating/steady/declining | Compare recent velocity to historical |

**Note**: GitHub API doesn't provide star history directly. Options:
1. Track stars over time in OPM database
2. Use GitHub Archive public dataset
3. Use star-history.com API (third-party)

### Repository Governance (v1.0)

| Check | API Endpoint |
|-------|--------------|
| Branch Protection | `/repos/{owner}/{repo}/branches/{branch}/protection` |
| Dependabot | Check for `.github/dependabot.yml` or repo settings |
| Secret Scanning | `/repos/{owner}/{repo}` (security settings) |
| Vulnerability Alerts | `/repos/{owner}/{repo}/vulnerability-alerts` |

### Fork Health Detection (Future)

The "OpenTofu problem": detecting when a fork from a license-changed upstream is becoming unsustainable.

**Detection Logic**:
1. Check if repo is a fork via `/repos/{owner}/{repo}` → `fork: true`, `parent`
2. Compare licenses: fork license vs upstream current license
3. If licenses differ, flag `license_diverged`
4. Track commits behind upstream via compare API
5. Monitor fork's own commit velocity over time

**Warning Signs**:
- License divergence (upstream changed to non-OSS)
- Rapidly growing commits behind (drift accelerating)
- Declining fork contributions (contributor fatigue)
- Few unique contributors to fork (bus factor risk)

**Example Detection**:
```json
{
  "fork_health": {
    "is_fork": true,
    "upstream_repo": "hashicorp/terraform",
    "license_diverged": true,
    "fork_license": "MPL-2.0",
    "upstream_license": "BSL-1.1",
    "commits_behind": 2847,
    "drift_velocity": 150,
    "fork_commit_trend": "declining"
  }
}
```

### GitLab Support (v1.x+)

Full GitLab integration including:
- GitLab group scanning
- GitLab-specific file patterns (.gitlab-ci.yml, merge request templates)
- GitLab API for metrics

### Responsiveness Metrics (v1.x)

| Metric | Description |
|--------|-------------|
| Issue Response Time | Median time to first response |
| PR Merge Time | Median time to merge |
| PR Review Time | Time from open to first review |

### Advanced Maintenance Metrics (v1.x)

| Metric | Description |
|--------|-------------|
| Release Cadence | Days since last release |
| Dependency Freshness | Outdated dependencies |
| Test Coverage | If available via badges/API |

### Package Health (Future)

| Metric | Description |
|--------|-------------|
| Dependency Freshness | Outdated dependencies count |
| Vulnerability Count | Open Dependabot/security alerts |
| Lock File Present | package-lock.json, go.sum, etc. |

### Integrations (v1.x+)

- GitLab support
- Zero Agent (CO agent skill)
- Backstage plugin
- Slack bot
- PagerDuty integration
