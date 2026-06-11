# AURAVYBE × meritstore — IAR handoff

**Role:** Inter-repo exchange SSOT — alpha customer requirements for **meritstore** (`AgentDraven/meritstore`).  
**Consumer:** AgentDraven implementers (separate repo/chat).  
**Validator:** Mr-PI-Bala · AURAVYBE_Portal (this repo).  
**Version:** 0.1.0-alpha · **Date:** 2026-06-11  
**Status:** Ready for AgentDraven intake (brainstorming phase complete)

**Governance:** This file is the **sole binding handshake** between AURAVYBE (requester) and AgentDraven meritstore (implementer) per MERIT L1 **§0.D IAR Code of Honor**. No side deals across repos or personas. Completion = requester sign-off in **§3.7** only.

---

## How to use this document

| Section | Purpose |
|---------|---------|
| **§1 PRD** | Normative product requirements |
| **§2 HLD** | Suggested design (AgentDraven may refine) |
| **§3 Acceptance** | Test cases and release gate before CTA cutover |
| **§4 Appendix** | Brainstorming rationale, pros/cons, deferred decisions |

**Code of Honor (MERIT L1 §0.D):** All asks, suggestions, scope changes, and acceptance criteria for this handoff live **only** in this document. AgentDraven must not treat chat, parallel docs, or verbal agreements as binding. Requester validates by checking off **§3.5** and signing **§3.7** — no integration cutover until **ACCEPT**.

**Next step:** AgentDraven implements **M0–M3** (Phase 1, Square). AURAVYBE runs §3 tests and signs off before changing marketing CTA.

---

## §1 PRD — Product requirements

### 1.1 Executive summary

AURAVYBE is the **first alpha customer** for **meritstore** — multi-tenant registration and payment at `meritstore.vercel.app`, routed by public slug (`/auravybe/register`).

**This repo:**

- Marketing SSOT (here.now + bit.ly today).
- Alpha requirements SSOT (this document).
- Tenant catalog authority via meritstore admin UI (not code deploys).

**Phase 1 payment (non-negotiable):** **Square** (verified EIN merchant). No Stripe or RevenueCat for alpha go-live.

### 1.2 Goals and non-goals

**Goals (alpha):**

1. Replace mailto registration with online registration + Square payment for June and July blocks.
2. AURAVYBE admin creates/edits blocks, tracks, capacity, pricing without deploys.
3. Parent flow: pick block(s) → student info → discounts → pay → confirmation.
4. Admin roster: registrations, payment status, CSV export.
5. Establish reusable tenant integration contract for SoulOS, DIRT, BioHack later.

**Non-goals (alpha):** RevenueCat, Stripe, mobile apps, full DIRT/SoulOS tier matrix, custom domain, migrating marketing off here.now.

### 1.3 Personas

| ID | Persona | Needs |
|----|---------|-------|
| **P1** | Parent / Guardian | Clear schedule, price, card pay, email confirmation |
| **P2** | Student | Named on registration; no login required for alpha |
| **P3** | AURAVYBE Admin (Mr. PI) | CRUD blocks, roster, export, comp/waitlist |
| **P4** | meritstore Platform Admin | Tenant onboarding, Square creds, observability |
| **P5** | Alpha Validator | Run §3 tests; sign off before production cutover |

### 1.4 Catalog and pricing (AURAVYBE seed)

| Rule | Value |
|------|--------|
| Daily rate | $45 / day |
| Standard block | 3 days → **$135** |
| Multi-block discount | **10%** (multiple blocks in one checkout) |
| Sibling discount | **10%**, stackable |
| June commitment | Full 3-day block per June offering |
| Location / BYOL | Brentwood; BYOLaptop + GitHub-for-Education acknowledgment |

**June 2026 blocks:**

| offering_id | Title | Dates | Time |
|-------------|-------|-------|------|
| `june-2026-b1` | Retro Arcade & AI Dungeon Master | Jun 16–18 | 9 AM – 12 PM |
| `june-2026-b2` | Magic 8-Ball Easter Egg Cheat Codes | Jun 23–25 | 9 AM – 12 PM |

**July 2026:** Two tracks per week — **Morning VYBE** (9–12) and **Afternoon AURA** (1–4). Admin defines weekly offerings and calendar dates. Marketing copy in `index.html` is descriptive; **meritstore catalog is authoritative**.

**Offering fields (minimum):** `offering_id`, `title`, `description_md`, `season`, `track`, `start_date`, `end_date`, times, `capacity`, `price_cents`, `status` (draft/open/full/closed), optional `square_catalog_object_id`.

### 1.5 User journeys

**P1 happy path:** Land on `/auravybe/register` → browse open offerings → cart → parent/student info → discounts → Square pay → confirmation → admin sees `paid`.

**P3 admin:** Sign in → CRUD offerings → set capacity/status → roster → CSV export.

**Failure paths:** capacity full (sold-out/waitlist); declined card (`pending_payment`, retry); webhook delay (idempotent handler); duplicate submit (idempotency key).

### 1.6 Tenant identity

| Field | Alpha value |
|-------|-------------|
| `tenant_id` / `public_slug` | `auravybe` |
| `display_name` | AURAVYBE |
| `payment_provider` | `square` |
| Supabase | Dedicated project recommended (`auravybe-meritstore`) |

### 1.7 Integration contract (normative behaviors)

| Surface | Purpose |
|---------|---------|
| `GET …/tenants/auravybe/offerings?status=open` | Parent catalog |
| `POST …/tenants/auravybe/registrations` | Create pending registration |
| `POST …/tenants/auravybe/checkout/square` | Start Square payment |
| `POST …/webhooks/square` | Payment confirmation |
| `GET …/tenants/auravybe/admin/roster` | Admin export (auth required) |

Exact paths are implementation details. Auth: Supabase for admin (`auravybe_admin`); parent identified by email. **Public slug routing only** — no API keys in URLs.

### 1.8 Success metrics

| Metric | Target |
|--------|--------|
| End-to-end registration + Square | All P0 tests pass |
| Admin publishes June Block 2 without deploy | Yes |
| Zero payment secrets in AURAVYBE_Portal git | Verified |
| Add one July week (AM+PM) | < 10 min via admin UI |

---

## §2 HLD — Suggested high-level design

### 2.1 System context

```text
bit.ly/auravybe → here.now (marketing) ──CTA──► meritstore.vercel.app/auravybe/register
                                                      │
                        ┌─────────────────────────────┼─────────────────────────────┐
                        ▼                             ▼                             ▼
                 Supabase (auravybe)            Square API + webhooks          (Future: RC, Stripe)
```

| Repo | Owner | Role |
|------|-------|------|
| `AgentDraven/meritstore` | Chief of Staff | Storefront, webhooks, admin UI, tenant registry |
| `AgentDraven/merit-private-vault` | Chief of Staff | Square + Supabase secrets |
| `Mr-PI-Bala/AURAVYBE_Portal` | Mr-PI-Bala | Marketing, this IAR doc, validation |

### 2.2 Suggested stack (alpha)

| Layer | Choice | Notes |
|-------|--------|-------|
| Host | Vercel **Hobby** | Alpha volume; upgrade at $5k/mo beta gate |
| App | Next.js or Astro + server routes | Webhooks need server |
| DB + Auth | Supabase (dedicated AURAVYBE project) | RLS, admin auth |
| Payments | **Square Web Payments SDK** + webhooks | Phase 1 only |
| Routing | `/{tenant_slug}/register`, `/{tenant_slug}/admin` | Multi-tenant registry |

### 2.3 Payment flow (Phase 1)

```text
Browser → meritstore → Supabase (registration: pending)
        → Square payment → webhook → Supabase (paid)
```

**PaymentProvider abstraction (suggested):** `SquareProvider` for alpha; stub `StripeProvider` / `RevenueCatProvider` for later.

### 2.4 Core data model (Supabase)

**offerings:** id, tenant_id, offering_id, title, description_md, season, track, dates, times, capacity, price_cents, status, square_catalog_id, timestamps.

**registrations:** id, tenant_id, offering_id FK, parent/student fields, status (`pending_payment` | `paid` | `cancelled` | `waitlist`), discount_cents, total_cents, square_payment_id, idempotency_key.

**admin_profiles:** user_id, tenant_id, role.

**RLS minimum:** public read open offerings; registrations via server routes; admin full CRUD for tenant.

### 2.5 API sketch

Base: `https://meritstore.vercel.app`

| Method | Path | Auth |
|--------|------|------|
| GET | `/api/v1/tenants/auravybe/offerings` | Public |
| POST | `/api/v1/tenants/auravybe/registrations` | Public |
| POST | `/api/v1/tenants/auravybe/checkout/square` | Public |
| POST | `/api/v1/webhooks/square` | Square signature |
| GET/POST/PATCH | `/api/v1/tenants/auravybe/admin/offerings` | Admin JWT |
| GET | `/api/v1/tenants/auravybe/admin/roster` | Admin JWT |

### 2.6 Discount engine (M3)

```text
base = sum(offering.price_cents)
multi_block = count(offerings) >= 2 ? base * 0.10 : 0
sibling = sibling_flag ? (base - multi_block) * 0.10 : 0
total = base - multi_block - sibling
```

Per-tenant config in tenant registry JSON (percentages, stackable flag).

### 2.7 Environments

| Env | Vercel | Supabase | Square |
|-----|--------|----------|--------|
| preview | PR branches | seed / branch | sandbox |
| production | main | auravybe prod | production |

Secrets in vault → Vercel env only. Sandbox acceptance before production CTA cutover.

### 2.8 Implementation phases (suggest to AgentDraven)

| Phase | Scope | AURAVYBE? |
|-------|--------|-----------|
| **M0** | Repo, tenant registry, `/auravybe/register` shell, Vercel deploy | Yes |
| **M1** | Supabase schema, admin CRUD, public catalog API | Yes |
| **M2** | Square checkout + webhook → `paid` | **Phase 1 go-live** |
| **M3** | Discounts, email confirmations | Yes |
| **M4** | Optional `/auravybe` marketing mirror | Later |
| **M5** | RevenueCat (other tenants) | No |
| **M6** | Stripe adapter | No |

**Alpha exit:** M0–M3 + §3 P0 pass.

### 2.9 Revenue-gated platform upgrades (all tenants)

| Stage | Trigger | Platforms |
|-------|---------|-----------|
| **Alpha** | Bootstrap | Vercel Hobby, Supabase Free, Square live |
| **Beta** | ≥ **$5,000/mo** | Vercel Pro, Supabase Pro, RC if needed |
| **Launch** | ≥ **$10,000/mo** | Production SLAs, custom domain, full RC |

### 2.10 MERIT guidelines for AgentDraven (L1 suggestions)

1. Two-surface model: marketing (here.now or `/tenant`) vs storefront (`/tenant/register`).
2. Central repo `AgentDraven/meritstore`; secrets in `env/meritstore/` and `env/tenants/{id}/`.
3. L3 product repos hold IAR docs only; no payment secrets in product git.
4. PaymentProvider adapter pattern; acceptance gate before tenant CTA cutover.
5. Document tenant registry in vault `docs/MERIT_STOREFRONT.md` when meritstore matures.

---

## §3 Acceptance tests and release gate

### 3.1 Gate rules

| Gate | Criteria |
|------|----------|
| **Sandbox pass** | All P0 in Square Sandbox + meritstore preview |
| **Production pass** | All P0 in production (small live or refund test) |
| **Sign-off** | Mr. PI marks §3.4 `ACCEPT` |
| **CTA cutover** | `index.html` register → `meritstore.vercel.app/auravybe/register` |

**Required report from AgentDraven:**

```markdown
## meritstore Alpha Test Report — auravybe
- Build / commit: …
- Environment: sandbox | production
- Date: …
| Test ID | PASS/FAIL | Evidence |
```

### 3.2 P0 — Must pass

**Catalog & routing**

| ID | Test | Expected |
|----|------|----------|
| T-P0-01 | Open `/auravybe/register` | Loads; tenant branding |
| T-P0-02 | Wrong slug | 404 or safe error |
| T-P0-03 | Catalog | June blocks with correct dates/times |
| T-P0-04 | Closed offering | Not selectable |

**Registration**

| ID | Test | Expected |
|----|------|----------|
| T-P0-10 | Single June block | → paid after Square |
| T-P0-11 | Two June blocks | 10% multi-block discount |
| T-P0-12 | Sibling flag | Additional 10% |
| T-P0-13 | At capacity | Sold-out/waitlist |
| T-P0-14 | Declined card | `pending_payment`, retry |
| T-P0-15 | Double-click submit | Idempotent; one paid row |

**Square & webhooks**

| ID | Test | Expected |
|----|------|----------|
| T-P0-20 | Sandbox success | `paid` within 60s |
| T-P0-21 | Webhook replay | No duplicate paid state |
| T-P0-22 | Bad signature | 401/403; no state change |

**Admin**

| ID | Test | Expected |
|----|------|----------|
| T-P0-30 | Admin login | Non-admin blocked from `/auravybe/admin` |
| T-P0-31 | Create July AM offering | Visible when `open` |
| T-P0-32 | Edit price/capacity | No redeploy needed |
| T-P0-33 | Roster CSV | Paid registrations included |

**Security**

| ID | Test | Expected |
|----|------|----------|
| T-P0-40 | No secrets in AURAVYBE_Portal | grep Square token → none |
| T-P0-41 | RLS | Parent cannot read others' registrations |
| T-P0-42 | here.now | Unchanged until sign-off |

### 3.3 P1 — Should pass (alpha+)

| ID | Test | Expected |
|----|------|----------|
| T-P1-01 | July AM+PM same week | Both registrable independently |
| T-P1-02 | Confirmation email | Parent receives summary |
| T-P1-03 | Mobile viewport | Usable on phone |
| T-P1-04 | Vercel analytics | View on `/auravybe/register` |

### 3.4 P2 — Out of scope

RevenueCat (Phase 3), Stripe checkout, marketing migration to meritstore (M4).

### 3.5 API comparison checklist

| PRD §1.7 requirement | Implemented? | Notes |
|----------------------|--------------|-------|
| Public catalog GET | ☐ | |
| Registration POST | ☐ | |
| Square checkout POST | ☐ | |
| Square webhook | ☐ | |
| Admin roster GET | ☐ | |
| Square only Phase 1 | ☐ | |

### 3.6 IAR tracker

| Date | Milestone | Status |
|------|-----------|--------|
| 2026-06-11 | IAR MERITSTORE.md published | Done |
| | meritstore M0 | Pending — AgentDraven |
| | M1 admin + catalog | Pending |
| | M2 Square | Pending |
| | M3 discounts | Pending |
| | Alpha sign-off | Pending |
| | CTA cutover | Pending |

### 3.7 Sign-off

```markdown
## AURAVYBE Alpha Acceptance
- Validator: Mr. PI
- Date: ___________
- meritstore commit: ___________
- Result: ACCEPT | REJECT
- Notes:
```

---

## §4 Appendix — Brainstorming and rationalization

*This section records decisions from the brainstorming phase so AgentDraven can see trade-offs considered. Not normative unless echoed in §1–§3.*

### 4.1 Two-surface architecture

| Option | Pros | Cons | Decision |
|--------|------|------|----------|
| **A: Single host (meritstore only)** | One deploy, one domain | Marketing loses here.now agent-fast publish | Rejected for alpha |
| **B: Marketing (here.now) + storefront (meritstore)** | Fast marketing iteration; payment isolation | Two URLs to maintain | **Chosen** |
| **C: Later merge marketing to `/auravybe` on meritstore** | Single domain eventually | Migration effort | Deferred M4 |

### 4.2 Payment provider

| Option | Pros | Cons | Decision |
|--------|------|------|----------|
| **Square Phase 1** | EIN account verified; painful to re-verify elsewhere | Less common in SaaS tutorials | **Chosen Phase 1** |
| **Stripe** | Strong dev UX, multi-tenant patterns | New business entity migration required | **Deferred M6+** |
| **RevenueCat for checkout** | Good for mobile subs | Wrong tool for one-time class blocks | **Deferred M5+ (subs only)** |

**Clarification:** Supabase holds registration state; **Square moves money** in Phase 1. RevenueCat is for subscription entitlements (SoulOS, DIRT), not alpha lab registration.

### 4.3 Hosting tier (Vercel)

| Stage | Rationale |
|-------|-----------|
| **Hobby for alpha** | Negligible traffic; bootstrapping |
| **Pro at ~$5k/mo** | Align upgrade with Supabase Pro + RC when revenue justifies |
| **Launch at ~$10k/mo** | Production discipline across stack |

Low volume alpha on Hobby is intentional, not an oversight.

### 4.4 Multi-tenant data separation

**Supabase:** One org → many **projects** (isolated DB per tenant). Preferred for privacy-heavy products. Free tier ≈ 2 projects account-wide — plan Pro or extra orgs as tenants grow.

**RevenueCat:** One account → multiple **projects/apps**. Separate RC project per product (AURAVYBE vs DIRT vs SoulOS) recommended. **Not used in alpha.**

**Shared DB + `tenant_id`:** Acceptable only with strict RLS; dedicated project preferred for AURAVYBE alpha.

### 4.5 Repo boundaries

| Choice | Rationale |
|--------|-----------|
| `AgentDraven/meritstore` implements | Chief of Staff owns multi-tenant platform |
| `AURAVYBE_Portal` = alpha customer only | Avoids storefront code sprawl in product repo |
| IAR doc in `AURAVYBE docs/IAR/` | Inter-repo strategic handoff per MERIT §0.D (no side deals) |
| `AURAVYBE_Portal` = marketing portal exemplar | MERIT L1 §0.E — poster child for `{Product}_Portal` pattern |

### 4.6 Routing and auth

Public slug (`/auravybe/register`) is fixed. Cross-platform auth (Supabase admin vs parent email) is **implementation detail** for AgentDraven — behaviors in §1.5 are normative.

### 4.7 What AURAVYBE does after handoff

1. **Wait** for AgentDraven meritstore M0–M3 delivery + test report.
2. **Run** §3 P0 (and P1 where possible) against sandbox then production.
3. **Sign** §3.7 if API matches §1.7 checklist.
4. **Cut over** marketing CTA only after acceptance.
5. **Do not** implement meritstore in this repo.

---

## Document control

| Version | Date | Notes |
|---------|------|-------|
| 0.1.0-alpha | 2026-06-11 | Consolidated IAR handoff (PRD + HLD + acceptance + appendix) |

**References:** `index.html` (marketing copy), `AURAVYBE.instructions` (L3), vault `AgentDraven/merit-private-vault`.
