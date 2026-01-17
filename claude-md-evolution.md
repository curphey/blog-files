# How My CLAUDE.md Evolved During a Site Migration

A before/after comparison showing how a CLAUDE.md file transformed from generic instructions into genuine project institutional memory.

## The Context

I'm migrating [crashoverride.com](https://crashoverride.com) from Next.js + PayloadCMS (hosted on Vercel) to a Hugo static site (hosted on Netlify). The goal: pixel-perfect replication. The complication: the repo already contained fragments from a previous failed migration attempt.

## The Pattern That Worked

1. **Describe the project to Claude first** — full context before any code
2. **Have Claude generate the CLAUDE.md** — it understood the problem better than I could articulate
3. **Add it to the repo and run `/init`** — now every session starts with that context
4. **Iterate as you learn** — each edge case and lesson goes back into the file

---

## BEFORE: Initial Version

This is what Claude generated after I explained the migration project and uploaded the implementation plan:

```markdown
# CLAUDE.md

This file provides guidance to Claude Code when working with this repository during and after the crashoverride.com website migration.

## Migration Context

**Source:** Next.js + PayloadCMS site hosted on Vercel (crashoverride.com)
**Target:** Hugo-based static site monorepo hosted on Netlify
**Goal:** Pixel-perfect replica of the existing site with improved architecture

This CLAUDE.md is structured to support the phased migration defined in `IMPLEMENTATION_PLAN.md`. After migration completes, this file should be updated to reflect the final architecture.

---

## Critical Requirements

> **These requirements are non-negotiable and apply to ALL work.**

### 1. Pixel-Perfect Visual Fidelity

The migrated Hugo site MUST be visually identical to crashoverride.com. This means:

- **Exact color matching** - Use scraped design tokens, not approximations
- **Exact typography** - Same fonts, sizes, weights, line heights, letter spacing
- **Exact spacing** - Margins, padding, gaps must match precisely
- **Exact component styling** - Buttons, cards, forms, navigation must be indistinguishable
- **Responsive behavior** - Mobile/tablet breakpoints must match the original exactly

**Validation:** Side-by-side comparison at each breakpoint (mobile, tablet, desktop) should show no visible differences.

### 2. GitHub Project Synchronization

GitHub issues and the project board must always reflect the exact current state of development.

---

## Architecture

### Monorepo Structure

├── sites/
│   ├── www/              # Main Hugo site (active - migration target)
│   ├── portal/           # Future: Customer portal
│   ├── admin/            # Future: Admin interface
│   └── api/              # Future: Zero RAG API
├── shared/               # Shared layouts, assets, images
├── scripts/              # Build, migration, and sync automation
│   ├── audit/            # Site audit scripts
│   ├── migration/        # Content migration scripts
│   └── sync-docs.sh      # Documentation sync
├── docs/
│   └── brandguidelines.md  # Design tokens and guidelines
└── IMPLEMENTATION_PLAN.md  # Phased migration plan

---

## Tech Stack

| Component | Technology | Version |
|-----------|------------|---------|
| Static Site Generator | Hugo Extended | 0.146.0+ |
| Search | Pagefind | Latest |
| Hosting | Netlify | N/A |
| Go | Go | 1.21+ |
| Node.js | Node.js | 20+ |

---

## Testing & Validation

# Validate Hugo configuration
cd sites/www && hugo config

# Check for broken links (requires lychee)
lychee --verbose --no-progress './sites/www/public/**/*.html'

# Spell check (requires typos)
typos ./sites/www/content

# HTML validation (requires html5validator)
html5validator --root ./sites/www/public

---

## CI/CD Pipeline

The CI workflow (`.github/workflows/ci.yml`) runs:

1. **Build validation** - Hugo build with production environment
2. **Link checking** - lychee checks for broken links
3. **Spell checking** - typos checks content
4. **HTML validation** - html5validator checks generated HTML
5. **Lighthouse audit** - Performance, accessibility, SEO (PRs only)

---

## Troubleshooting

### Hugo Module Errors

cd sites/www
hugo mod clean
hugo mod tidy

### Build Failures

# Clear caches
rm -rf sites/www/public sites/www/resources
cd sites/www && hugo mod clean && hugo mod tidy

# Verbose debug
hugo --verbose --debug

---

## Post-Migration Updates

After the migration is complete, update this CLAUDE.md to:

1. Remove migration-specific sections
2. Add documentation for the final architecture
3. Update component documentation
4. Add content authoring guidelines
5. Document any custom shortcodes created
```

**What was good:** Clear structure, pixel-perfect requirement called out, basic commands documented.

**What was missing:** No awareness of the dual codebase problem, no content migration rules, no quality standards, no tooling for validation.

---

## AFTER: Evolved Version

After multiple iterations as we hit edge cases and refined our approach:

```markdown
# CLAUDE.md

This file provides guidance to Claude Code when working with this repository during and after the crashoverride.com website migration.

## Migration Context

**Source:** Next.js + PayloadCMS site hosted on Vercel (crashoverride.com)
**Target:** Hugo-based static site monorepo hosted on Netlify
**Goal:** Pixel-perfect replica of the existing site with improved architecture

### Repository State

This repository contains multiple codebases that need to be merged:

| Location | Description |
|----------|-------------|
| `sites/www/` | **Target** - Existing Hugo-based developer hub site |
| `oldsite/` | **Source** - Next.js + PayloadCMS marketing site to be migrated |
| Various locations | Fragments from a previous failed migration attempt |

**⚠️ WARNING: Previous Migration Artifacts**

This repo may contain fragments from a previous failed migration attempt. When working on the migration:
- Do not assume existing code in unexpected locations is correct or complete
- Verify any existing migration scripts or partial conversions before using them
- When in doubt, refer to the source in `oldsite/` or the live site at crashoverride.com
- Clean up orphaned migration artifacts as you encounter them

### Planning & Tracking

- **Implementation Plan:** `IMPLEMENTATION_PLAN.md` (in repo root)
- **GitHub Issues:** https://github.com/crashappsec/xxx-xxx/issues
- **Project Board:** crashappsec/projects/xxx

All work should follow the phased approach in the implementation plan and be tracked via GitHub issues.

### PayloadCMS API Access

The following curl command has been validated for extracting content from the PayloadCMS backend:

```bash
curl -g 'https://xxx.com/api/blogs' \
  -H 'Authorization: users API-Key <REDACTED>'
```

**Available API endpoints** (append to `https://xxx.com/api/`):
- `blogs` - Blog posts
- `pages` - Static pages
- `media` - Media assets

---

## Critical Requirements

> **These requirements are non-negotiable and apply to ALL work.**

### 1. Pixel-Perfect Visual Fidelity

The migrated Hugo site MUST be visually identical to crashoverride.com. This means:

- **Exact color matching** - Use scraped design tokens, not approximations
- **Exact typography** - Same fonts, sizes, weights, line heights, letter spacing
- **Exact spacing** - Margins, padding, gaps must match precisely
- **Exact component styling** - Buttons, cards, forms, navigation must be indistinguishable
- **Responsive behavior** - Mobile/tablet breakpoints must match the original exactly

**Validation:** Side-by-side comparison at each breakpoint (mobile, tablet, desktop) should show no visible differences.

### 2. Exact Content Migration (No Modifications)

> **⚠️ CRITICAL: MIGRATE CONTENT EXACTLY AS-IS**
>
> ALL content and media from PayloadCMS MUST be migrated **exactly as it exists** with **NO modifications whatsoever**:
> - **NO editing** of text, titles, or copy
> - **NO reformatting** of content structure
> - **NO summarizing** or condensing
> - **NO "improvements"** to grammar, spelling, or style
> - **NO optimization** of images beyond format conversion (if needed)
> - **NO renaming** of files or assets
> - **NO omitting** of any content, even if it seems outdated or redundant
>
> The migration is a **transfer**, not an editorial process. Content decisions are made by humans, not during migration.

### 3. GitHub Project Synchronization

GitHub issues and the project board must always reflect the exact current state of development.

**Proactive Sync Requirements:**
1. **Before starting work:** Check the relevant issue exists and is assigned to the correct milestone
2. **During work:** Update issue status on the project board to "In Progress"
3. **After completing work:** Close issues immediately with summary comments
4. **When blocked:** Add comments explaining blockers and update status

### 4. No Unsolicited UI/UX Changes

**DO NOT** make any unsolicited web UI or UX changes. All design and styling decisions are handled by ensuring visual parity with the source site. Your role during migration is to **replicate**, not redesign.

### 5. HTML & CSS Quality Standards

**HTML Standards:**
- Valid HTML5 (W3C validation with zero errors)
- Semantic markup (`<header>`, `<nav>`, `<main>`, `<article>`, `<section>`, `<footer>`)
- Proper heading hierarchy (no skipped levels)
- All images have alt attributes
- Form inputs have associated labels
- No inline styles

**CSS/SCSS Standards:**
- No `!important` (except documented utilities)
- BEM naming convention (`.block__element--modifier`)
- No ID selectors for styling
- No magic numbers - use `_scraped-tokens.scss` variables
- Maximum 3 levels of SCSS nesting

---

## Architecture

### Current Repository Structure (Pre-Migration)

├── sites/
│   ├── www/              # Existing Hugo developer hub (KEEP & EXTEND)
│   ├── portal/           # Future: Customer portal
│   ├── admin/            # Future: Admin interface
│   └── api/              # Future: Zero RAG API
├── oldsite/              # Next.js + PayloadCMS source (MIGRATE FROM)
│   ├── src/              # React components, pages
│   ├── payload/          # CMS collection definitions
│   ├── public/           # Static assets
│   └── ...               # Next.js configuration
├── shared/               # Shared layouts, assets, images
├── scripts/              # Build, migration, and sync automation
├── docs/
│   └── brandguidelines.md  # Design tokens and guidelines
├── IMPLEMENTATION_PLAN.md  # Phased migration plan
└── CLAUDE.md             # This file

### Merge Strategy

The migration involves **merging** two sites, not replacing one with the other:

| Content Type | Source | Target |
|--------------|--------|--------|
| Documentation (Chalk, Zero, Ocular) | `sites/www/` (existing Hugo) | `sites/www/content/docs/` |
| Marketing pages (homepage, pricing, about, etc.) | `oldsite/` (Next.js) | `sites/www/content/` |
| Blog posts | PayloadCMS API | `sites/www/content/blog/` |
| Design system | `oldsite/` styles | `sites/www/assets/scss/` |
| Components | `oldsite/src/components/` | `sites/www/layouts/partials/` |

**Key Principle:** The existing Hugo site in `sites/www/` is the foundation. Marketing content and styling from `oldsite/` gets migrated INTO it, not the other way around.

---

## Tech Stack

| Component | Technology | Version |
|-----------|------------|---------|
| Static Site Generator | Hugo Extended | 0.146.0+ |
| Search | Pagefind | Latest |
| Hosting | Netlify | N/A |
| Go | Go | 1.21+ |
| Node.js | Node.js | 20+ |

### Required: Claude Superpowers Plugin

When working on this migration, use the **Claude Superpowers browser plugin** for:

- **Visual comparison** - Side-by-side screenshot comparison of source vs migrated pages
- **Style extraction** - Capturing computed styles from the live site
- **DOM inspection** - Analyzing component structure on crashoverride.com
- **Responsive testing** - Validating breakpoint behavior matches the source

---

## Migration-Specific Guidelines

### Handling Previous Migration Artifacts

Before starting any migration task:

1. **Check for existing work** - Search the repo for files related to your task
2. **Validate existing code** - If you find migration code, verify it against `oldsite/` source
3. **Don't assume correctness** - Previous attempts may be incomplete or incorrect
4. **Clean as you go** - Remove orphaned files that won't be used
5. **Document decisions** - Note why you kept or discarded existing code

### React to Hugo Component Conversion

When converting React components from `oldsite/src/components/` to Hugo partials:
- The Hugo partial must produce **identical HTML output** to the React component
- Preserve all CSS classes, data attributes, and ARIA attributes
- Reference the original React component in `oldsite/` as the source of truth
- Test at all breakpoints after conversion
- Document any Hugo-specific adaptations needed

---

## Testing & Validation

# Validate Hugo configuration
cd sites/www && hugo config

# Check for broken links
lychee --verbose --no-progress './sites/www/public/**/*.html'

# Spell check
typos ./sites/www/content

# HTML validation
html5validator --root ./sites/www/public

# CSS/SCSS linting
npx stylelint "sites/www/assets/scss/**/*.scss"

# Accessibility audit
pa11y http://localhost:1313 --standard WCAG2AA

# Run all quality checks
npm run lint
npm run validate
npm run a11y

---

## CI/CD Pipeline

The CI workflow (`.github/workflows/ci.yml`) runs:

1. **Build validation** - Hugo build with production environment
2. **HTML validation** - html5validator checks generated HTML (zero errors required)
3. **CSS/SCSS linting** - stylelint checks for code quality and BEM compliance
4. **Link checking** - lychee checks for broken links
5. **Spell checking** - typos checks content
6. **Accessibility audit** - pa11y checks WCAG 2.1 AA compliance
7. **Lighthouse audit** - Performance, accessibility, SEO, best practices (≥85 required)

---

## Issue Acceptance Criteria Template

All issues should include these acceptance criteria:

### Functional
- [ ] Feature works as described
- [ ] Edge cases handled

### Visual Fidelity (Migration)
- [ ] Side-by-side comparison shows no visible differences
- [ ] Matches source at desktop (1440px), tablet (768px), mobile (375px)
- [ ] Animations/transitions match source timing

### HTML/CSS Quality
- [ ] HTML passes W3C validation (zero errors)
- [ ] Semantic HTML elements used appropriately
- [ ] All images have alt attributes
- [ ] No inline styles
- [ ] BEM naming convention followed
- [ ] All values use design tokens from _scraped-tokens.scss

### Accessibility
- [ ] Keyboard navigation works
- [ ] Focus indicators visible
- [ ] Color contrast meets WCAG AA (4.5:1 text, 3:1 large text)
- [ ] Screen reader tested

### Performance
- [ ] Lighthouse score ≥ 85 (all categories)
- [ ] No layout shift (CLS < 0.1)
- [ ] Images optimized and lazy-loaded

### Browser Compatibility
- [ ] Works in Chrome, Safari, Firefox (latest)
- [ ] No console errors

---

## Post-Migration Updates

After the migration is complete, update this CLAUDE.md to:

1. Remove migration-specific sections
2. Archive `oldsite/` directory
3. Update architecture documentation
4. Add content authoring guidelines
5. Document any custom shortcodes created
```

---

## What Changed: Summary

| Addition | Why It Mattered |
|----------|-----------------|
| **Repository State table** | Claude needed to know there were TWO codebases to merge, not one to replace |
| **Previous migration warning** | Stopped Claude from trusting orphaned code from failed attempts |
| **PayloadCMS API with curl command** | Exact command for content extraction, no guessing |
| **Merge Strategy table** | Clear mapping of what goes where |
| **Exact content migration rules** | Prevented Claude from "helpfully" editing content during transfer |
| **Claude Superpowers plugin** | Tooling for visual regression testing |
| **HTML/CSS quality standards** | BEM naming, semantic HTML, no inline styles, SCSS nesting limits |
| **Expanded acceptance criteria** | Comprehensive checklist for every PR |
| **Handling previous artifacts section** | Process for dealing with legacy code |

---

## The Key Insight

The first version was a reasonable starting point. But a CLAUDE.md isn't a one-and-done artifact — it's a living document that captures what you learn as the project progresses.

Every time we hit an edge case ("wait, there's old migration code in here"), discovered a constraint ("content must transfer exactly as-is"), or found useful tooling ("use the Superpowers plugin for visual comparison"), that knowledge went back into the file.

The pattern: **Context first → Generate → Commit → Iterate**

Your CLAUDE.md isn't documentation. It's institutional memory for your AI pair programmer.
