# Website Migration Implementation Plan

This document outlines the implementation plan for migrating and consolidating the Crash Override website and developer hub. Each section represents a GitHub issue to be created upon approval.

---

## Phase 0: Infrastructure & Deployment (Do First)

Get the current developer hub code deployed and running before migration work begins.

### Issue #1: Set Up Netlify Account and Site
**Labels:** `phase-0`, `infrastructure`, `priority-critical`

**Description:**
Create and configure the Netlify account and site for crashoverride.com.

**Tasks:**
- [ ] Create Netlify team/account (or verify existing)
- [ ] Create site for crashoverride.com
- [ ] Configure team permissions
- [ ] Set up billing/plan appropriate for traffic
- [ ] Document account credentials securely

**Acceptance Criteria:**
- Netlify account created and accessible
- Team members have appropriate access

---

### Issue #2: Configure Netlify Deployment Pipeline
**Labels:** `phase-0`, `infrastructure`, `priority-critical`

**Description:**
Set up multi-environment deployment with GitHub Actions integration.

**Tasks:**
- [ ] Connect GitHub repo to Netlify
- [ ] Configure build settings for Hugo
- [ ] Set up branch deploys (main → production, staging, develop)
- [ ] Configure deploy previews for PRs
- [ ] Set up custom domains (crashoverride.com, staging.crashoverride.com, dev.crashoverride.com)
- [ ] Configure SSL/TLS certificates
- [ ] Set up build hooks for content updates
- [ ] Create GitHub Actions workflow for CI/CD
- [ ] Test full deployment pipeline with current code

**Acceptance Criteria:**
- Current developer hub code deploys successfully
- All environments (prod, staging, dev) accessible
- PR previews working
- GitHub Actions pipeline functional

---

### Issue #3: Configure GitHub Actions CI/CD
**Labels:** `phase-0`, `infrastructure`, `priority-critical`

**Description:**
Set up GitHub Actions workflows for automated testing, building, and deployment.

**Tasks:**
- [ ] Create `.github/workflows/ci.yml` for linting and testing
- [ ] Create `.github/workflows/deploy-preview.yml` for PR previews
- [ ] Create `.github/workflows/deploy-production.yml` for production deploys
- [ ] Configure Netlify auth tokens as GitHub secrets
- [ ] Set up Lighthouse performance audits on PRs
- [ ] Set up broken link checker
- [ ] Test workflows on feature branches

**Acceptance Criteria:**
- CI runs on all PRs
- Deployments trigger automatically on branch pushes
- Preview URLs posted to PRs

---

## Phase 1: Foundation

### Issue #4: Create Monorepo Structure
**Labels:** `phase-1`, `infrastructure`, `priority-high`

**Description:**
Set up the monorepo directory structure as defined in the roadmap.

**Tasks:**
- [ ] Create `sites/www/` directory structure for main Hugo site
- [ ] Create `sites/portal/` placeholder for future portal SPA
- [ ] Create `sites/admin/` placeholder for future admin interface
- [ ] Create `sites/api/` placeholder for Zero RAG API
- [ ] Create `shared/` directory for shared layouts, assets, and static files
- [ ] Create `scripts/` directory for build and migration scripts
- [ ] Update `.github/workflows/` for new structure

**Acceptance Criteria:**
- Directory structure matches roadmap specification
- README updated with structure overview

---

### Issue #5: Configure Hugo Site with Subdirectories
**Labels:** `phase-1`, `hugo`, `priority-high`

**Description:**
Set up unified Hugo site with `/docs` and `/developers` subdirectories for SEO consolidation.

**Tasks:**
- [ ] Configure Hugo base configuration in `config/_default/`
- [ ] Set up content directories: `content/docs/` and `content/developers/`
- [ ] Configure Hugo mounts for shared partials
- [ ] Set up environment-specific configs (development, staging, production)
- [ ] Update `netlify.toml` with environment contexts

**Acceptance Criteria:**
- Hugo builds successfully with all environments
- `/docs` and `/developers` routes work correctly
- Environment-specific settings apply correctly

---

### Issue #6: Set Up Shared Components Directory
**Labels:** `phase-1`, `hugo`, `priority-medium`

**Description:**
Create shared components structure for reusable layouts, partials, and assets.

**Tasks:**
- [ ] Create `shared/layouts/partials/` for header, footer, navigation
- [ ] Create `shared/assets/scss/` for shared styles
- [ ] Create `shared/assets/js/` for shared JavaScript
- [ ] Create `shared/static/images/` for logos and icons
- [ ] Configure Hugo module mounts to include shared components

**Acceptance Criteria:**
- Shared partials can be used across all content sections
- SCSS compiles correctly with shared tokens

---

### Issue #7: Scrape crashoverride.com Styles
**Labels:** `phase-1`, `design`, `priority-high`

**Description:**
Extract design tokens and styles from the current crashoverride.com site to ensure visual consistency.

**Tasks:**
- [ ] Create `scripts/scrape-styles.js` script
- [ ] Extract color palette (primary, secondary, accent colors)
- [ ] Extract typography settings (fonts, sizes, weights)
- [ ] Extract spacing scale
- [ ] Extract component styles (buttons, cards, forms)
- [ ] Generate `_scraped-tokens.scss` file
- [ ] Document extracted design tokens

**Acceptance Criteria:**
- SCSS variables file generated with all tokens
- Documentation of design system created

---

### Issue #8: Configure HTTP Security Headers
**Labels:** `phase-1`, `security`, `priority-high`

**Description:**
Set up comprehensive security headers in Netlify configuration.

**Tasks:**
- [ ] Configure Content-Security-Policy header
- [ ] Configure X-Frame-Options (DENY)
- [ ] Configure X-Content-Type-Options (nosniff)
- [ ] Configure Referrer-Policy
- [ ] Configure Permissions-Policy
- [ ] Configure Cross-Origin policies (COOP, COEP, CORP)
- [ ] Test headers with securityheaders.com

**Acceptance Criteria:**
- All security headers configured in `netlify.toml`
- Security score of A or A+ on securityheaders.com

---

### Issue #9: Set Up TLS/HSTS
**Labels:** `phase-1`, `security`, `priority-high`

**Tasks:**
- [ ] Verify Netlify automatic TLS certificates
- [ ] Configure HSTS header with preload flag
- [ ] Verify force HTTPS redirect is enabled
- [ ] Submit to HSTS preload list (hstspreload.org)

**Acceptance Criteria:**
- TLS 1.2+ enforced
- HSTS preload submission complete

---

## Phase 2: Main Site Migration

### Issue #10: Automated Site Audit Script
**Labels:** `phase-2`, `migration`, `priority-high`

**Description:**
Create automated scripts to audit the existing crashoverride.com site.

**Tasks:**
- [ ] Create `scripts/audit/` directory
- [ ] Build crawler script to discover all pages/routes
- [ ] Extract and catalog all URLs with response codes
- [ ] Identify page types (blog, docs, static, etc.)
- [ ] Detect CMS-managed vs static content patterns
- [ ] Inventory all media assets (images, PDFs, videos) with sizes
- [ ] Detect third-party integrations (scripts, iframes, APIs)
- [ ] Identify React components by analyzing JS bundles
- [ ] Generate JSON audit report
- [ ] Generate human-readable markdown summary

**Acceptance Criteria:**
- Script runs automatically against live site
- Outputs complete inventory as JSON and markdown
- Identifies all URLs for redirect mapping

---

### Issue #11: Define Canonical Content Schemas
**Labels:** `phase-2`, `migration`, `priority-high`

**Description:**
Define clean content schemas for Hugo, breaking from Payload CMS legacy structure.

**Tasks:**
- [ ] Define BlogPost schema (title, slug, date, content, author, categories, tags)
- [ ] Define DocPage schema (title, section, weight, content, toc)
- [ ] Define StaticPage schema (title, layout, content)
- [ ] Define allowed values (categories, doc sections)
- [ ] Create schema transformation mappings from Payload
- [ ] Document schema in TypeScript interfaces

**Acceptance Criteria:**
- All content schemas documented
- Transformation logic defined for each Payload field

---

### Issue #12: Build Content Migration Pipeline
**Labels:** `phase-2`, `migration`, `priority-high`

**Description:**
Create scripts to migrate content from Payload CMS and live site.

**Tasks:**
- [ ] Create `scripts/migration/` directory structure
- [ ] Implement Payload CMS API export script
- [ ] Implement schema transformation layer
- [ ] Implement HTML to Markdown converter
- [ ] Implement site scraper for non-CMS content (Puppeteer)
- [ ] Create content validation script
- [ ] Build migration CLI wrapper script

**Acceptance Criteria:**
- Scripts can export all Payload content
- Content transforms to Hugo-compatible format
- Validation catches missing/invalid fields

---

### Issue #13: Media Audit and Cleanup
**Labels:** `phase-2`, `migration`, `priority-medium`

**Description:**
Audit media files, remove orphans, and optimize assets.

**Tasks:**
- [ ] Create media audit script to scan all files
- [ ] Identify referenced vs orphaned media
- [ ] Identify duplicate files (by hash)
- [ ] Identify oversized files (>2MB)
- [ ] Implement dry-run cleanup mode
- [ ] Implement image optimization (resize, compress)
- [ ] Generate cleanup report

**Acceptance Criteria:**
- Report generated showing orphaned/duplicate/oversized files
- Only referenced media migrated to new site
- Images optimized (max 1920px width)

---

### Issue #14: Run Content Migration
**Labels:** `phase-2`, `migration`, `priority-high`

**Description:**
Execute the content migration and validate results.

**Tasks:**
- [ ] Set up Payload CMS API credentials
- [ ] Run Payload content export with schema transformation
- [ ] Run site scraper for non-CMS content
- [ ] Run media cleanup (dry-run first, then live)
- [ ] Migrate only referenced media to `static/uploads`
- [ ] Update internal links to new URL structure
- [ ] Run content validation, fix errors
- [ ] Generate migration reports for review

**Acceptance Criteria:**
- All content migrated to Hugo format
- No validation errors
- Migration report reviewed and approved

---

### Issue #15: Convert React Components to Hugo Partials
**Labels:** `phase-2`, `hugo`, `priority-high`

**Description:**
Convert existing React components to Hugo partial templates.

**Tasks:**
- [ ] Convert navigation/header component
- [ ] Convert footer component
- [ ] Convert hero sections
- [ ] Convert card components
- [ ] Convert CTA components
- [ ] Convert form components (using custom forms, not HubSpot embeds)
- [ ] Convert any custom widgets

**Acceptance Criteria:**
- All major components converted
- Visual parity with current site

---

### Issue #16: Set Up URL Redirects
**Labels:** `phase-2`, `seo`, `priority-high`

**Description:**
Configure redirects for old URLs to maintain SEO value.

**Tasks:**
- [ ] Map old URLs to new URL structure (from audit data)
- [ ] Configure redirects in `netlify.toml`
- [ ] Set up Hugo aliases in frontmatter where appropriate
- [ ] Test all redirects

**Acceptance Criteria:**
- All old URLs redirect to new locations
- No 404s for previously valid URLs

---

### Issue #17: Implement SEO Configuration
**Labels:** `phase-2`, `seo`, `priority-high`

**Description:**
Set up hierarchical SEO configuration (global, section, page levels).

**Tasks:**
- [ ] Set up global SEO defaults in `config/_default/params.toml`
- [ ] Configure section-level SEO for `/blog`, `/docs`, `/developers`
- [ ] Implement SEO cascade logic in Hugo partials
- [ ] Add OpenGraph meta tags
- [ ] Add Twitter Card meta tags
- [ ] Add schema.org JSON-LD templates

**Acceptance Criteria:**
- SEO meta tags render correctly at all levels
- Social sharing previews work correctly

---

### Issue #18: Implement HubSpot API Integration
**Labels:** `phase-2`, `analytics`, `priority-high`

**Description:**
Integrate with HubSpot using their APIs instead of embedded forms for full control over UX.

**Tasks:**
- [ ] Set up HubSpot API credentials (private app)
- [ ] Create `lib/hubspot.ts` API client
- [ ] Implement Forms API submission endpoint
- [ ] Implement Contacts API for direct contact creation/updates
- [ ] Build custom form components with our own styling
- [ ] Create serverless function (Netlify Functions) for API proxy
- [ ] Implement form validation (client and server)
- [ ] Set up error handling and retry logic
- [ ] Create Hugo shortcodes for embedding custom forms
- [ ] Test all form submissions flow to HubSpot

**Acceptance Criteria:**
- Custom forms submit to HubSpot via API
- Contact records created/updated in HubSpot
- No HubSpot embed code on site
- Full control over form UX

---

### Issue #19: Set Up Analytics and Tracking
**Labels:** `phase-2`, `analytics`, `priority-medium`

**Description:**
Configure analytics tools (excluding HubSpot forms, covered separately).

**Tasks:**
- [ ] Set up Google Analytics 4
- [ ] Set up Google Tag Manager
- [ ] Integrate HubSpot tracking code (analytics only)
- [ ] Configure data layer for GTM
- [ ] Implement cookie consent banner (GDPR)
- [ ] Test tracking in staging environment

**Acceptance Criteria:**
- GA4 tracking events firing correctly
- Cookie consent compliant with GDPR

---

## Phase 2.5: Marketing Pages

### Issue #20: Build Global Site Elements
**Labels:** `phase-2.5`, `ui`, `priority-high`

**Description:**
Build shared global elements for the marketing site.

**Tasks:**
- [ ] Build header/navigation with mobile menu
- [ ] Build footer with link structure
- [ ] Create 404 error page
- [ ] Implement global site search (Pagefind)
- [ ] Add skip navigation link (accessibility)
- [ ] Implement ARIA landmarks

**Acceptance Criteria:**
- Header/footer render on all pages
- Mobile navigation works correctly
- Site search functional

---

### Issue #21: Build Homepage
**Labels:** `phase-2.5`, `pages`, `priority-high`

**Description:**
Build the main marketing homepage.

**Tasks:**
- [ ] Hero section with primary CTA
- [ ] Logo bar (customer/partner logos)
- [ ] Pain points section
- [ ] Features overview
- [ ] Social proof section (testimonials, stats)
- [ ] Secondary CTAs
- [ ] SEO optimization

**Acceptance Criteria:**
- Homepage matches design specs
- All sections responsive
- CTAs link correctly

---

### Issue #22: Build Product/Platform Page
**Labels:** `phase-2.5`, `pages`, `priority-high`

**Tasks:**
- [ ] Platform overview section
- [ ] How it works section
- [ ] Features grid/list
- [ ] Architecture diagram
- [ ] Demo request CTA (custom form → HubSpot API)

**Acceptance Criteria:**
- Page matches design specs
- Demo CTA submits to HubSpot via API

---

### Issue #23: Build Pricing Page
**Labels:** `phase-2.5`, `pages`, `priority-high`

**Tasks:**
- [ ] Pricing tier comparison table
- [ ] Annual/monthly toggle
- [ ] Feature comparison matrix
- [ ] FAQ section
- [ ] CTAs for each tier

**Acceptance Criteria:**
- Pricing toggle works correctly
- CTAs link to appropriate actions

---

### Issue #24: Build About/Company Pages
**Labels:** `phase-2.5`, `pages`, `priority-medium`

**Tasks:**
- [ ] Company story section
- [ ] Values section
- [ ] Timeline/milestones
- [ ] Leadership team bios
- [ ] Investors section
- [ ] Press mentions

**Acceptance Criteria:**
- All sections complete
- Team photos optimized

---

### Issue #25: Build Careers Page
**Labels:** `phase-2.5`, `pages`, `priority-medium`

**Tasks:**
- [ ] Culture section
- [ ] Benefits overview
- [ ] DEI statement
- [ ] Job listings integration (Greenhouse/Lever)
- [ ] Application form/link

**Acceptance Criteria:**
- ATS integration working
- Job listings display correctly

---

### Issue #26: Build Security/Compliance Page
**Labels:** `phase-2.5`, `pages`, `priority-medium`

**Tasks:**
- [ ] Certifications display (SOC2, etc.)
- [ ] Security architecture overview
- [ ] Trust center link
- [ ] Security whitepaper download
- [ ] Contact security team form (custom → HubSpot API)

**Acceptance Criteria:**
- All certifications displayed
- Whitepaper download works

---

### Issue #27: Build Solutions Pages Template
**Labels:** `phase-2.5`, `pages`, `priority-medium`

**Description:**
Create reusable template for persona and industry solutions pages.

**Tasks:**
- [ ] Create solutions page template
- [ ] For Security Teams page
- [ ] For Developers page
- [ ] For DevOps page
- [ ] By Industry templates (Financial Services, Healthcare, SaaS)

**Acceptance Criteria:**
- Template is reusable
- All persona pages created

---

### Issue #28: Build Customer Content Hub
**Labels:** `phase-2.5`, `pages`, `priority-medium`

**Tasks:**
- [ ] Customers hub page with logo grid
- [ ] Filtering by industry/use case
- [ ] Aggregate metrics section
- [ ] Case study template
- [ ] Testimonial carousel component
- [ ] G2/Gartner review widget integration

**Acceptance Criteria:**
- Logo grid displays correctly
- Case studies render from markdown

---

### Issue #29: Build Resources Section
**Labels:** `phase-2.5`, `pages`, `priority-low`

**Tasks:**
- [ ] Events/Webinars calendar page
- [ ] Registration integration (custom form → HubSpot API)
- [ ] Past recordings archive
- [ ] Support/Help Center page
- [ ] Integrations marketplace page

**Acceptance Criteria:**
- Event calendar functional
- Integration cards display correctly

---

### Issue #30: Build Partners Page
**Labels:** `phase-2.5`, `pages`, `priority-low`

**Tasks:**
- [ ] Partner program overview
- [ ] Partner tiers explanation
- [ ] Partner directory
- [ ] Success stories
- [ ] Partner application form (custom → HubSpot API)

**Acceptance Criteria:**
- Application form submits to HubSpot via API

---

### Issue #31: Implement Accessibility Compliance
**Labels:** `phase-2.5`, `accessibility`, `priority-high`

**Description:**
Ensure WCAG AA compliance across all pages.

**Tasks:**
- [ ] Audit color contrast ratios
- [ ] Add alt text to all images
- [ ] Ensure keyboard navigation works
- [ ] Test with screen readers
- [ ] Add focus indicators
- [ ] Implement skip navigation
- [ ] Add ARIA labels where needed

**Acceptance Criteria:**
- WCAG AA compliance verified
- No critical accessibility issues

---

## Phase 3: Authentication & Email

### Issue #32: Set Up Supabase Project
**Labels:** `phase-3`, `infrastructure`, `priority-high`

**Tasks:**
- [ ] Create Supabase project
- [ ] Configure authentication providers
- [ ] Set up database tables (see schema in roadmap)
- [ ] Configure Row Level Security policies
- [ ] Set up storage buckets
- [ ] Document environment variables

**Acceptance Criteria:**
- Supabase project configured
- RLS policies in place

---

### Issue #33: Configure SendGrid for Email
**Labels:** `phase-3`, `email`, `priority-high`

**Tasks:**
- [ ] Set up SendGrid account
- [ ] Configure DNS records (SPF, DKIM, DMARC)
- [ ] Create email templates for invitations
- [ ] Implement email sending service
- [ ] Test email deliverability

**Acceptance Criteria:**
- Emails send successfully
- DNS authentication passes

---

### Issue #34: Implement Authentication Flows
**Labels:** `phase-3`, `auth`, `priority-high`

**Tasks:**
- [ ] Implement Google OAuth for admin (domain-restricted to crashoverride.com)
- [ ] Implement magic link flow for portal users
- [ ] Create auth callback handlers
- [ ] Implement session management
- [ ] Add logout functionality

**Acceptance Criteria:**
- Admin login restricted to company domain
- Magic links work for portal users

---

## Phase 4-8: Future Phases (Placeholder Issues)

### Issue #35: Multi-Portal System (Phase 4)
**Labels:** `phase-4`, `feature`, `priority-medium`

**Description:**
Build the multi-tenant portal system with dynamic routing.

*Detailed tasks to be added when Phase 4 begins.*

---

### Issue #36: Admin Interface (Phase 5)
**Labels:** `phase-5`, `feature`, `priority-medium`

**Description:**
Build the internal admin interface for content and user management.

*Detailed tasks to be added when Phase 5 begins.*

---

### Issue #37: Design System Migration (Phase 6)
**Labels:** `phase-6`, `design`, `priority-low`

**Description:**
Migrate from scraped styles to Figma design tokens.

*Detailed tasks to be added when Phase 6 begins.*

---

### Issue #38: Zero RAG API (Phase 7)
**Labels:** `phase-7`, `api`, `priority-low`

**Description:**
Build the Zero RAG API with rate limiting and usage tracking.

*Detailed tasks to be added when Phase 7 begins.*

---

### Issue #39: Automatic Localization (Phase 8)
**Labels:** `phase-8`, `i18n`, `priority-low`

**Description:**
Implement multi-language support with automatic translation.

*Detailed tasks to be added when Phase 8 begins.*

---

## GitHub Labels to Create

| Label | Color | Description |
|-------|-------|-------------|
| `phase-0` | `#0052CC` | Phase 0: Infrastructure (Do First) |
| `phase-1` | `#1D76DB` | Phase 1: Foundation |
| `phase-2` | `#5319E7` | Phase 2: Main Site Migration |
| `phase-2.5` | `#7057FF` | Phase 2.5: Marketing Pages |
| `phase-3` | `#0E8A16` | Phase 3: Auth & Email |
| `phase-4` | `#FBCA04` | Phase 4: Multi-Portal |
| `phase-5` | `#F9A825` | Phase 5: Admin Interface |
| `phase-6` | `#E65100` | Phase 6: Design System |
| `phase-7` | `#D73A49` | Phase 7: Zero RAG API |
| `phase-8` | `#B60205` | Phase 8: i18n |
| `infrastructure` | `#C5DEF5` | Infrastructure/DevOps |
| `hugo` | `#FF6F00` | Hugo static site |
| `migration` | `#D4C5F9` | Content migration |
| `seo` | `#0366D6` | SEO related |
| `security` | `#B60205` | Security related |
| `design` | `#F9D0C4` | Design/UI |
| `ui` | `#BFD4F2` | UI components |
| `pages` | `#C2E0C6` | Page content |
| `analytics` | `#FEF2C0` | Analytics/tracking |
| `accessibility` | `#D4C5F9` | Accessibility |
| `auth` | `#FBCA04` | Authentication |
| `email` | `#E99695` | Email/notifications |
| `api` | `#1D76DB` | API development |
| `i18n` | `#006B75` | Internationalization |
| `priority-critical` | `#000000` | Critical priority |
| `priority-high` | `#B60205` | High priority |
| `priority-medium` | `#FBCA04` | Medium priority |
| `priority-low` | `#0E8A16` | Low priority |

---

## Summary

| Phase | Issues | Priority | Description |
|-------|--------|----------|-------------|
| **Phase 0: Infrastructure** | #1-3 | Critical | Get current code deployed first |
| Phase 1: Foundation | #4-9 | High | Monorepo, Hugo, security |
| Phase 2: Main Site Migration | #10-19 | High | Audit, migrate, convert, SEO, HubSpot API |
| Phase 2.5: Marketing Pages | #20-31 | Medium-High | All marketing pages |
| Phase 3: Auth & Email | #32-34 | Medium | Supabase, SendGrid, OAuth |
| Phases 4-8 | #35-39 | Future | Portals, admin, design, API, i18n |

**Total Issues:** 39

---

## Key Changes from Original

1. **Phase 0 added** - Netlify account setup and deployment pipeline comes first
2. **Issue #10 (was #7)** - Site audit is now automated via scripts, not a spreadsheet
3. **Issue #18** - HubSpot integration uses APIs instead of embedded forms
4. **All form references** - Updated to use custom forms → HubSpot API pattern

---

## Next Steps

1. Review this implementation plan
2. Approve or request changes to issues
3. Create GitHub issues from approved items
4. Begin Phase 0 (get current site deployed)
 laude
