# Lunchkit

**Working domain:** lunchkit.app
**Build target:** Claude Code, working directly against this project's GitHub repository. This file lives at the repo root as `CLAUDE.md` and is read automatically as project instructions the moment Claude Code opens the repo — no re-explaining context in chat.
**Document status:** Build-ready
**Last updated:** 2026-08-26

---

## Project Overview & Core Problem

Lunchkit is a multi-sided platform that removes all the coordination friction from company-subsidized catered lunches.

**Core problem it solves:**
- Caterers currently suffer from inaccurate headcounts, last-minute changes, wasted food, late payments, and zero direct feedback.
- Companies want to offer lunch as a benefit but hate the spreadsheet tracking, partial employee contributions, and payment chasing.
- Employees just want an easy way to see what's for lunch, know exactly what they will pay, and order with one or two taps.

Lunchkit makes the entire loop self-service:
- Caterer publishes menu + sets per-company plate price + sets cutoff.
- Company sets a fixed dollar subsidy per meal and manages employees.
- Employee sees clear price ("Company pays $X, you pay $Y"), orders before cutoff, and pays only their share.
- Employee share goes to the caterer immediately via Stripe.
- Company is invoiced cleanly for their subsidy portion + any extra plates.
- Pickup is confirmed via static QR + fallbacks so everyone stays honest.
- No-shows are automatically tracked so waste is visible.

The product is sold primarily to caterers as a SaaS tool. Companies are brought on by the caterer.

---

## Target Users & Jobs-to-Be-Done

### A. Caterer Admin (primary customer)
**Jobs:**
- Publish accurate weekly (or multi-week) menus with almost no ongoing admin.
- Get reliable paid headcounts per company and in total.
- Receive employee payments immediately.
- Know which companies are good/paying customers.
- Hand a clean packing/delivery list to their driver or kitchen helper.
- See issue reports from employees (wrong item, allergen, missing component, etc.).
- Invite and manage the companies they serve.
- Invite other members to the Caterer organization and assign roles.

### B. Caterer Delivery / Kitchen role (limited)
**Jobs:**
- See only the daily packing & delivery summary (counts + delivery notes).
- Cannot see money, pricing, or edit anything.

### C. Company Full Admin
**Jobs:**
- Onboard / deactivate employees easily (with clear handling of future paid orders).
- Set the fixed company subsidy amount per meal.
- Black out holidays and special non-delivery days.
- Manage extra/buffer plates (default + day-specific) and allocate them to employees or guests.
- See usage, pickups, and no-shows (waste visibility).
- Promote or demote other employees to Full Admin or Lunch Coordinator.
- Note: Every Full Admin is also a normal Employee and can order lunch themselves.

### D. Company Lunch Coordinator ("Lunch Lady")
**Jobs:**
- See today's orders + pickup status (employees and named guests).
- Mark people (and named guests) as received (with logging).
- Handle proxy pickups when one employee picks up for another.
- Call over people who haven't collected yet.
- Cannot change prices, employees, or money settings.
- Note: Also a normal Employee who can order lunch.

### E. Employee
**Jobs:**
- See the menu and exact out-of-pocket price.
- Order (or skip) before the clear cutoff.
- Pay only their share with one tap.
- Confirm they received their meal (QR, self-confirm, or one-time proxy code).
- Report an issue with a meal (goes to caterer).
- Receive helpful, non-annoying reminders.
- See any company-provided (earmarked extra) meals already covered for them.

### F. Developer (platform-level)
**Jobs:**
- View and manage all feedback and bug tickets submitted by any user.
- Update ticket status as it's worked.
- Leave internal notes on tickets.
- Basic visibility into recent signups and active caterers.

---

## Core Features (Prioritized: MVP vs. Later)

### MVP (Must ship first)

**Auth, Identity & Roles**
- Single login UI for everyone. After login the system routes by role.
- Permanent login URL: `lunchkit.app/login`
- Login methods:
  - Email + password
  - Microsoft SSO (work account; email must match the invited work email)
  - Biometric / passkey unlock for returning users (fingerprint, Face ID, device biometrics) for near-instant daily access
- Magic-link + 6-digit code invite pattern used for:
  - Caterer → Company Admin invite
  - Company → Employee invite
  - Caterer Admin → other Caterer members
- **Company side:** Every person is an Employee first. Full Admin and Lunch Coordinator are permissions layered on top. First Company Admin claims invite, sets password, becomes Full Admin + Employee. Clear guidance to use personal work email.
- **Caterer side (mirrored model):** Every person is a member of the Caterer organization first. Roles: Caterer Admin (full) and Caterer Delivery/Kitchen (limited). The person who signs up the Caterer becomes the first Caterer Admin and can later invite others and assign roles.
- Soft activate / deactivate for employees.
- Admin users (both Company and Caterer) default to their normal daily experience, with a clear persistent way to open Admin tools.

**Hard business rule – One meal per employee per day**
- An employee may have at most **one active order per calendar day**.
- No quantity selector in the UI.
- Backend enforces uniqueness on (user_id + date).
- If an order already exists for that day, the employee can cancel and re-order a different meal (before cutoff) but can never hold two concurrent orders for the same day.
- No ordering on behalf of other people. Each person uses their own account.

**Inactivity & Auto-deactivation**
- Default: employee is soft-deactivated after **30 days** with no login and no order activity.
- Company Full Admins can set the company-wide window between **30 and 90 days**.
- Full Admins never auto-deactivate.
- Optional **Protected / Executive** flag on any employee → 180-day inactivity window.
- Deactivated user who tries to log in sees a clear message + **"Request reactivation"** button that notifies all Company Full Admins.
- Only **active** employees count toward the soft employee ceilings used for caterer billing.
- **Deactivation with future paid orders:**
  - Admin sees a confirmation modal listing upcoming paid orders and the consequence.
  - On confirm: future orders are cancelled (caterer does not cook them).
  - Employee's paid share is converted to **account credit** (visible to the employee and usable after reactivation).
  - No automatic cash refund in MVP.

**Caterer**
- Meal library: create once (photo, name, short description, optional allergens), reuse forever via search/dropdown.
- **Composite meals:** a single meal offering can contain multiple library items (e.g. Main + Rolls + Salad). Published and ordered as one unit.
- Build menus up to 8+ weeks out.
- Assign any meal to one or more companies.
- Set different plate price per company.
- Set order cutoff (default 72 hours, fully customizable per caterer).
- Live confirmed paid headcounts (per company + grand total).
- Daily packing / delivery summary (counts + optional delivery notes).
- Basic "This week" snapshot: confirmed plates, expected employee revenue, outstanding company balances, no-shows.
- One-click copy of a previous week's menu as starting point.
- Receive and view **Report issue** submissions from employees (with optional photo).
- Invite Company flow.
- Invite other Caterer members and assign Admin or Delivery roles.

**Company**
- Add employees (first name, last name, work email, optional employee code).
- Set fixed dollar subsidy per meal.
- Blackout calendar:
  - Pre-populated standard U.S. holidays (New Year's Day, Memorial Day, Independence Day, Labor Day, Thanksgiving, Christmas, etc.) that can be toggled on/off individually.
  - Ability to add custom one-off or recurring blackout dates.
- **Extra / buffer plates:**
  - Optional default daily buffer (e.g. "always 5 extra").
  - Ability to add extra plates for specific days only.
  - Any extra plate can be left open or allocated to:
    - An existing employee (they see the day as company-provided, no charge to them), or
    - A free-text guest name (consultant, temp, visitor — no login).
  - Simple calendar / list overview showing extras per day and current allocations.
- View today's orders, pickups, and **no-shows**.
- Simple usage / waste visibility.
- Manage roles (promote/demote Full Admin and Coordinator).
- Set Protected / Executive flag and inactivity window.

**Employee**
- See only the days/menus the caterer has made visible.
- Extremely clear cutoff date/time on every order screen.
- See exact split: "Company pays $X • You pay $Y" (or "Company provided" if earmarked extra).
- One-tap order + pay (Stripe, card on file) — one meal per day maximum.
- See any company-provided (earmarked extra) days already covered for them.
- Static company QR code for pickup confirmation.
- In-app "I received my meal" fallback button.
- **One-time "Pick up for me" code:** employee with an unpicked-up order can generate a single-use code/link that expires at the end of the lunch window. Another person can use it at the station so the original employee is marked received. Logged as proxy pickup.
- **Report issue** on an order (reason + optional photo) → goes to the Caterer.
- Smart notifications: approaching cutoff, lunchtime reminder, "did you get it?".
- First-login install guidance (choose iPhone or Android → exact steps).
- Install instructions always available later under Profile / Settings.
- Account credit balance visible if they have credits from cancelled future orders.

**Lunch Coordinator**
- Sees employees with orders/allocations and their pickup status.
- Sees named guests (no app) for the day and can mark them received manually.
- Can see and process proxy "Pick up for me" codes.
- Clear "Already picked up" state if someone tries to scan again.

**No-show handling**
- After the company's defined lunch window ends, any order that was never marked received is automatically set to **No-show**.
- Visible to Company Admin and Caterer for waste tracking and conversation.

**Feedback & Bugs System (standard feature — build-support infrastructure, not a nice-to-have)**
- Available to every logged-in user via Profile / menu ("Send feedback"). This is how Taylor talks to Claude about the app during and after the build, instead of re-explaining notes in chat every session.
- Type: Bug / Feature Idea / Note. Rich-text title + description (bold, italics, bullet points) — both the description and any screenshots are optional independently; only the type is required.
- Optional screenshot upload (1–3 images).
- Submits a `FeedbackItem` with status `Pending` that appears in the Developer Portal.
- Separate from the meal-specific **Report issue** flow (which goes to the Caterer, not the Developer Portal — different audience, different purpose).

**Developer Portal**
- Platform-level role (not tied to any Caterer or Company).
- Support for one or multiple Developer users.
- List of all `FeedbackItem`s with filters (type, status, date).
- Ticket detail view (description, screenshots, submitter, organization).
- Status workflow: **Pending** → **Resolved**, **Deferred**, **Decided Not To**, or **Fixed** (bugs resolve specifically as *Fixed*; feature ideas and notes resolve as *Resolved*, *Deferred*, or *Decided Not To*).
- Ability to leave internal notes on tickets.
- Basic list of recent signups / active caterers.
- Because the backend is Supabase (see Tech Stack), a Claude Code session working on the app should query the `FeedbackItem` table directly at the start of a build or maintenance session to pick up open bugs/ideas/notes — see Implementation Notes.

**Payments**
- Employee pays only their share at order time → funds go to caterer via Stripe Connect.
- Company is invoiced weekly (or on chosen cadence) for (subsidy × confirmed/picked-up meals) + extra plates.
- Company can pay inside the app or download clean PDF invoice for AP.
- Card data is handled entirely by Stripe (never stored by Lunchkit).
- Account credits from cancelled future orders are applied to future employee shares.

**Platform & Marketing Site**
- High-quality Progressive Web App (installable on iOS & Android home screen).
- Full responsive web experience optimized for desktop (Caterer & Company Admin).
- Single high-converting marketing page at lunchkit.app (detailed specification in the Marketing Site appendix).
- Dedicated minimal invite/claim pages (never the marketing homepage).
- Clear "Log in" button on marketing site → `lunchkit.app/login`.
- No dead ends: unknown email on login → friendly message + path to correct signup/claim.
- First-login install guidance for **all** roles.
- **Legal entity & branding:**
  - Website footer: `© 2026 Clique Studios IO. All rights reserved.`
  - In-app About section (under Profile / Settings): `Lunchkit is a product of Clique Studios IO` + link to `https://cliquestudios.io`
  - Stripe subscription charges to caterers appear on statements as **Clique Studios IO** (not Lunchkit).
- **Share Lunchkit** (available to every user, near Feedback): simple action that opens the native share sheet or copies a clean link to the marketing page (`lunchkit.app`). Optional light pre-filled text. No tracking or rewards in MVP.

### Later (Phase 2)
- 5-star ratings (public or per-meal) + optional short comment on every meal.
- Advanced dashboards & exportable reports (including waste / no-show trends).
- Ingredient lists / shopping estimates.
- Predictive headcount based on historical participation.
- Multi-location support per company.
- Native App Store / Play Store wrappers (Capacitor).
- Formal caterer referral program (refer another caterer → free month after they pay).
- Caterer can message companies inside the app.
- Advanced crash / performance monitoring (e.g. Sentry).
- **Multi-choice meals**: Caterer can optionally offer up to 2–3 meal choices per company per day. Employees see the available options and pick one (still limited to one selection per day). Headcounts, packing lists, and reporting break down by choice.
- **Caterer Directory ("Find a Caterer")**: public, opt-in directory on the marketing site so companies without a caterer (or looking to switch) can find active Lunchkit caterers near them and request a quote. Doubles as a lead-gen/retention hook for the Caterer (primary customer). Full detail immediately below.

**Caterer Directory ("Find a Caterer") — Feature Detail**

*Origin: raised by Vanessa in the Clique Studios board meeting — companies who hear about Lunchkit but don't yet have (or want to switch) a caterer should be able to find one who's already on the platform.*

**Why:** Some companies discover Lunchkit before they have a caterer, or their current caterer isn't on it. This gives them a self-serve way to find one who is, and gives caterers a built-in lead-gen channel simply for being active Lunchkit subscribers.

**Caterer-side (Profile settings)**
- New "Directory Listing" toggle in Caterer Admin settings — **opt-in, off by default** (see flagged decision below).
- When enabled, the caterer fills in: display business name, city, state, optional short blurb, and a **Directory Contact Email** — which the caterer can set independently of their login email, specifically for fielding inbound leads.
- Toggle can be switched off at any time; the listing disappears from the public directory immediately.

**Eligibility**
- Only caterers with an **active, paying subscription** (not trialing, not past-due/canceled) can appear, even if opted in.
- Listing auto-hides the moment a subscription lapses and auto-reappears if it becomes active again — no manual re-opt-in required.

**Public directory page (marketing site)**
- New page, e.g. `lunchkit.app/caterers` ("Find a Caterer"), linked from the marketing nav/footer.
- Company visitor filters by state (and optionally city) to browse profile cards: business name, city/state, blurb.
- No caterer email address is ever printed directly on the public page — avoids scraping and spam. Each card instead has a **"Request Catering"** button.

**Request Catering flow (lead form, not a raw mailto link)**
1. Company visitor clicks "Request Catering" on a caterer's card.
2. Short form: their name, work email, company name, approximate employee count, target per-plate budget, desired meals per week, optional message.
3. On submit: Lunchkit emails the caterer's Directory Contact Email with the submitted details, reply-to set to the requester's address, and stores a `CatererLead` record for the caterer's own reference.
4. The caterer replies directly to the requester by email to continue the conversation — Lunchkit does not broker or track the rest of that relationship.
5. Basic spam protection required (rate limiting + honeypot or CAPTCHA) since this is a public, unauthenticated form.

**Open decision for Taylor/Vanessa:** this plan defaults to **opt-in** (caterer must actively enable listing) rather than auto-listing every active subscriber, since some caterers may not want to be publicly discoverable (already at capacity, competitive sensitivity, etc.). If the board wants it default-on for all active subscribers instead, that's a one-line change to the default — flagging it here so it's a conscious choice rather than an assumption baked into the build.

### Delight Layer (Later — after core product is stable)
A fun, non-blocking personality system designed to bring unexpected joy and make people smile when they open the app.

**Food Character Cast**
- A cast of silent food characters (carrot, broccoli, pear, spinach leaf, etc.).
- Each character has a distinct personality and role archetype (e.g. the overworked one, the slacker, the boss, the gossip, the rule-follower).
- Characters never speak or use thought bubbles — all communication is pure physical comedy and gesture.
- Different visual styling and scene sets for Employee side vs Caterer side (office/field life vs kitchen/chef life).

**Scene Behavior**
- Short animated scenes (typically 5–30 seconds) that appear as lightweight overlays.
- Appear completely randomly and infrequently (a user might see one and not see another for weeks or months).
- Scenes auto-dismiss; they never permanently block the UI.
- Some scenes are purely observational (characters walk on, interact with each other, walk off).
- Some scenes are lightly interactive (a character reaches for a UI element; if the user taps it, a surprise consequence happens — e.g. something spills, the character gets drenched, then reacts and exits).
- Strict rules: never interrupt ordering, payment, QR scanning, or any critical path. Respect `prefers-reduced-motion`.

**Seasonal & Holiday Variants**
- Special scene packs that only rotate during defined seasons or around the standard holiday list (Thanksgiving table, winter bar with snow, summer lounge chair, etc.).
- Because companies use the shared standard holiday list, seasonal scenes can be timed consistently across the platform.

**Controls**
- Global on/off switch (platform level).
- Ability to retire individual scenes from rotation.
- Future possibility: light industry flavor if companies declare an industry at setup.

**Intent**
- Make the app feel alive and human.
- Give employees a tiny reason to smile or look forward to opening it.
- Create mild character identification ("that's so me").
- Never at the expense of clarity, speed, or trust in the core lunch-ordering experience.

This layer is intentionally deferred until the core product is proven and stable.

---

## Detailed User Flows

### Caterer First-Time Signup
1. Lands on lunchkit.app → "I'm a Caterer" / "Start free" → Get started.
2. Signs up with business name, personal name, email, password.
3. Becomes the first Caterer Admin of the new Caterer organization.
4. Lands in Caterer onboarding (Stripe Connect, default cutoff, first meals).
5. 30-day free trial begins automatically.
6. First-login install guidance is shown.
7. From dashboard can "Invite Company" and "Invite team member".

### Caterer → Company Invite & Claim
1. Caterer enters company name + admin's work email.
2. System creates Company record under that Caterer and sends invite email (magic link + short code).
3. Recipient clicks link → lands on dedicated claim page.
4. Clear guidance shown: "Use your personal work email. This creates your employee profile and gives you Full Admin rights. You can later give admin access to anyone else."
5. They set their own password → become Full Admin + Employee.
6. First-login install guidance is shown.
7. They can now set subsidy, invite employees, and manage roles.

### Employee Invite & First Login
1. Company Full Admin creates employee (name, work email, optional employee code).
2. System emails magic link + 6-digit code.
3. Employee clicks → dedicated invite page → enters code (or pre-filled) → sets password (or uses Microsoft SSO).
4. Immediately after first successful auth: short "Get Lunchkit on your phone" step.
5. Subsequent opens: biometric / passkey unlock preferred.
6. Install instructions remain available under Profile / Settings.
7. Admin has Resend + Copy invite text buttons.

### Employee Ordering Flow (One Meal Per Day)
1. Opens app → sees upcoming days with clear cutoff badges.
2. Taps a day → sees meal (possibly composite), exact price or "Company provided".
3. If no existing order: "Order & Pay $Y".
4. If already ordered: "Already ordered" / option to cancel and change (before cutoff only).
5. Backend rejects any second order for the same user + date.
6. After cutoff the day is locked.

### One-time "Pick up for me" Flow
1. Employee with an unpicked-up order for today generates a single-use code/link.
2. Code expires at the end of the company's lunch window.
3. Another person uses the code at the station (or shows it to the Lunch Coordinator).
4. Original employee is marked received; action is logged as proxy pickup by the helper.
5. Repeat scan after already picked up shows clear "Already picked up" state.

### Extra Plate Allocation Flow
1. Company Admin sets default buffer and/or adds extras for specific days.
2. Admin can allocate any extra plate to an employee or enter a guest name.
3. Employee who is allocated sees the day as already covered.
4. Lunch Coordinator sees named guests and marks them received manually.

### Pickup Confirmation Flow
1. Preferred: Employee scans the permanent company QR → marked received.
2. Fallback: "I received my meal" button (available from lunch start until end of day + buffer).
3. Proxy: one-time "Pick up for me" code.
4. Lunch Coordinator can mark employees or named guests received (logged).
5. After lunch window: any still-unmarked order auto-becomes **No-show**.

### Report Issue Flow
1. Employee (or Coordinator) opens an order → "Report issue".
2. Selects reason, optionally adds photo and short note.
3. Submitted to the Caterer for that day's orders.
4. Separate from app-level "Send feedback".

### Deactivation with Future Orders
1. Full Admin chooses to deactivate an employee who has future paid orders.
2. Modal lists the orders and states: "These orders will be cancelled. The employee's paid share will become account credit."
3. On confirm: orders cancelled, credit issued, employee deactivated.
4. Reactivation restores access and makes credit available.

### Feedback Submission Flow (App)
1. Any logged-in user → Profile / menu → "Send feedback".
2. Type (Bug / Feature Idea / Note), title, rich text, optional screenshots.
3. A `FeedbackItem` is created with status `Pending` and appears in the Developer Portal.

### Caterer Directory Lead Flow (Phase 2)
1. Company visitor (no login required) lands on `lunchkit.app/caterers`, filters by state/city.
2. Taps "Request Catering" on a caterer card → short lead form (company name, contact name/email, estimated headcount, target per-plate budget, desired meals/week, optional message).
3. Submits → Lunchkit emails the caterer's Directory Contact Email with the lead details (reply-to = requester's email) and logs a `CatererLead` record tied to that caterer.
4. Caterer follows up directly with the requester by email/phone; Lunchkit's role ends at delivering the lead.
5. Caterer can review past leads for their listing under Directory settings (simple chronological list; no in-app messaging in Phase 2).

---

## UI/UX Principles (including specific places tooltips and guidance should appear)

**Overall principles**
- Calm, clean, generous whitespace.
- Zero guilt language.
- Extremely clear hierarchy and status.
- Progressive disclosure: only show what the current role needs.
- Mobile-first for employees; desktop-comfortable for admins.
- Admin rights feel like an elevated layer, never a separate product.
- Delete every possible point of friction (install, login, daily unlock, pickup).

**Critical guidance locations (tooltip / inline-help required)**
- Company claim page: use personal work email.
- First login: install steps (iPhone vs Android).
- Price split explainer on first view.
- Cutoff badge always visible and impossible to miss.
- QR / proxy / already-picked-up states crystal clear.
- Company-provided day: "Company provided – no charge".
- Deactivation confirmation modal when future orders exist.
- Deactivated login: clear message + request reactivation.
- No-show visibility for admins and caterers.
- Directory listing toggle (Phase 2): explain that turning it on makes the caterer's business name, city/state, and blurb publicly visible on the marketing site.

---

## Recommended Tech Stack (optimized for low cost, without sacrificing function, speed, or UX)

- **Framework:** Next.js 15 (App Router)
- **Language:** TypeScript
- **Styling:** Tailwind CSS + shadcn/ui
- **Auth:** Clerk (multi-tenant + Microsoft SSO + passkeys/biometrics)
- **Database:** Supabase (Postgres). *Flagged stack call:* this swaps in for a generic Neon/plain-Postgres setup now that the build has no Replit dependency — it matches the backend Taylor already runs elsewhere, and it lets a Claude Code session query app data (including the `FeedbackItem` table) directly via the Supabase MCP connector instead of through a UI. Easy to swap back to Neon + plain Postgres if preferred.
  - Enable Row Level Security on every multi-tenant table, scoped by `caterer_id` / `company_id`. An RLS-enabled table with no policies yet blocks all access by default — that's the safe starting state while policies are being written, not a bug.
- **ORM:** Prisma, pointed at the Supabase Postgres connection string.
- **Payments:** Stripe + Stripe Connect
- **Email:** Resend
- **File storage:** Supabase Storage (or Cloudflare R2 if a non-Supabase option is preferred)
- **Push notifications:** Web Push (VAPID)
- **PWA:** next-pwa or built-in Next.js PWA support
- **Hosting:** Vercel (Next.js-native, generous free tier, deploys straight from the GitHub repo's `staging` branch for preview builds and `main` for production)
- **Source control:** GitHub. Claude Code builds directly against this repo — see Git Workflow Rules below.

---

## Database Schema Outline

- **User** (id, email, name, auth_id, created_at)
- **Caterer** (id, name, stripe_account_id, default_cutoff_hours, trial_ends_at, plan, directory_opt_in, directory_contact_email, directory_city, directory_state, directory_blurb, ...)
- **CatererMembership** (id, caterer_id, user_id, role: admin | delivery, active, ...)
- **Company** (id, caterer_id, name, subsidy_amount_cents, inactivity_days, lunch_window_end, ...)
- **CompanyMembership** (id, company_id, user_id, role: full_admin | coordinator | employee, employee_code, active, protected_executive, last_activity_at, credit_balance_cents, ...)
- **Meal** (id, caterer_id, name, description, photo_url, allergens, ...)
- **MealComponent** (id, meal_id, library_item_id, ...)
- **MenuItem** (id, company_id, meal_id, date, plate_price_cents, published, ...)
- **Order** (id, menu_item_id, user_id, employee_share_cents, status: ordered | paid | picked_up | no_show | cancelled, source: employee | company_extra, stripe_payment_id, ...)
  → Unique constraint on (user_id, date) for active employee orders
- **ExtraPlate** (id, company_id, date, allocated_to_user_id nullable, guest_name nullable, status, ...)
- **PickupProxyCode** (id, order_id, code, created_by, used_by nullable, expires_at, used_at nullable)
- **PickupLog** (id, order_id or extra_plate_id, method: qr | self | coordinator | proxy, marked_by, created_at)
- **IssueReport** (id, order_id, reporter_id, reason, note, photo_url, created_at, ...)
- **Blackout** (id, company_id, date, recurring_yearly, note, ...)
- **Invoice** (id, company_id, period_start, period_end, amount_cents, status, pdf_url, ...)
- **Invite** (id, type, target_email, code, status, expires_at, ...)
- **Subscription** (id, caterer_id, plan, status, current_period_end, ...)
- **FeedbackItem** (id, user_id, type: bug | feature_idea | note, title, description, status: pending | resolved | deferred | decided_not_to | fixed, created_at, ...) — the Feedback & Bugs System; a Claude Code session queries this directly (see Implementation Notes).
- **FeedbackScreenshot** (id, feedback_item_id, image_url, created_at)
- **DeveloperNote** (id, feedback_item_id, author_id, note, created_at)
- **ReactivationRequest** (id, company_membership_id, status, created_at, ...)
- **CatererLead** (id, caterer_id, requester_name, requester_email, company_name, estimated_headcount, budget_per_plate_cents, meals_per_week, message, created_at) — Phase 2, feeds the Caterer Directory "Request Catering" flow.

All queries scoped by caterer_id or company_id.

---

## Key Screens / Pages

**Marketing** — lunchkit.app (see the Marketing Site appendix)
- Phase 2: `/caterers` ("Find a Caterer") — public directory + Request Catering lead form.

**Auth & Invites**
- /login (email/password, Microsoft SSO, biometrics)
- /invite/company/[code], /invite/employee/[code], /invite/caterer-member/[code]

**Employee**
- Home / Upcoming days
- Day detail + order
- My orders / history / credits
- Scan QR + "Pick up for me" code generation
- Report issue
- Profile / Install / Send feedback / Share Lunchkit / About

**Company Full Admin**
- Dashboard (usage, no-shows, waste signals)
- Employees + roles + Protected/Executive + inactivity settings
- Subsidy & settings
- Blackout calendar
- Extra plates calendar / allocation
- Invoices

**Company Lunch Coordinator**
- Today's board (employees + guests + proxy status)
- Mark received

**Caterer Admin**
- Home snapshot
- Meal library + composite builder
- Menu planner
- Companies + Invite
- Team members
- Packing lists
- Issue reports from employees
- Billing
- Phase 2: Directory listing settings + received leads

**Caterer Delivery** — packing/delivery summary only

**Developer Portal** — feedback/bug tickets + basic activity

---

## Edge Cases & Error States

- Second order same day → blocked.
- After cutoff → locked with clear message.
- QR when no order → "You don't have a meal ordered today."
- Already picked up → clear "Already picked up" state.
- Proxy code expired or already used → clear error.
- Late self-confirm still allowed within window.
- Company blackout → ordering disabled + reason.
- Deactivated login → message + request reactivation.
- Deactivation with future paid orders → mandatory confirmation modal + credit path.
- Stripe failure → order not created; clear retry.
- Last Full Admin cannot demote themselves.
- Auto no-show after lunch window.
- Guest name only → Coordinator marks by name.
- Directory: caterer's subscription lapses → listing auto-hidden until active again, no manual re-opt-in needed.
- Directory: no caterers match the visitor's filters → friendly empty state, suggest broadening the search.
- Directory: lead form abuse → rate-limited + honeypot/CAPTCHA; excessive submissions blocked.

---

## Suggested Pricing / Monetization

*(Finalized — use these figures directly, not placeholders.)*

**Primary customer = Caterer**

| Plan     | Active Companies | Base Monthly | Employees included (per company) |
|----------|------------------|--------------|----------------------------------|
| Starter  | 1                | $79          | Up to 250                        |
| Growth   | 2–3              | $149         | Up to 250 each                   |
| Scale    | 4–6              | $249         | Up to 250 each                   |
| Custom   | 7+               | Contact us   | —                                |

**Soft overages (per company)**
0–250 included · 251–400 +$29 · 401–600 +$59 · 601+ custom

Only **active** employees count.

**Trial:** 30-day free trial for every new caterer.
**Annual:** Pay for 10 months, get 12.

Employees never pay for the software. Companies are not charged by Lunchkit in v1.

---

## Git Workflow Rules
- ALWAYS commit directly to the staging branch.
- DO NOT create a new feature branch when saving or committing changes (unless staging has not yet been created, in which case create that branch on the first commit).
- It is explicitly allowed and expected to commit directly to the staging branch.
- DO NOT commit to the main or master branches. Never commit to either of those branches.

---

## Testing Checklist Protocol
At the end of every response where you've made or changed code, include a short "Test this" checklist — 2 to 5 concrete things Taylor should click through or verify — plus a prompt for her to report back any bugs or feedback before you continue building.

---

## Implementation Notes for the Coding Agent

This file is `CLAUDE.md` at the repo root — Claude Code reads it automatically as project context the moment the repo is opened, no re-pasting needed. Work through the build order below in sequence, committing to the `staging` branch per the Git Workflow Rules above as each numbered item is completed, and closing every response with the Testing Checklist Protocol above.

**Build order (strict)**
1. Auth + multi-tenant models (including Microsoft SSO + passkeys/biometrics)
2. Caterer signup + trial + Invite Company + Invite Caterer member
3. Meal library + composite meal builder + menu assignment
4. Employee invite + ordering (one-meal-per-day) + Stripe Connect
5. Extra plates + allocation (employee or guest) + overview
6. Cutoff + blackout (standard holidays)
7. Pickup confirmation (QR + self + coordinator + proxy "Pick up for me")
8. Auto no-show after lunch window
9. Report issue (to caterer, with photo)
10. Inactivity / auto-deactivation + reactivation requests + deactivation-with-credit flow
11. Company subsidy invoicing + PDF
12. Role management + Admin overlay UX
13. Feedback & Bugs System (`FeedbackItem`) + Developer Portal
14. Role-specific dashboards and packing lists
15. Billing / plan management
16. Marketing landing page
17. PWA + first-login install guidance + Profile install section
18. Polish empty/loading/error states and mobile UX

This strict order covers MVP only. Phase 2 and Delight Layer features (Core Features section) are sequenced after item 18 is stable, and — unlike the MVP list — have no fixed internal order relative to each other unless a dependency requires it. The Caterer Directory has no MVP dependencies beyond an existing base of paying caterer subscribers, so it can be picked up any time after MVP ships; it does depend on Subscription status being reliable (item 15) since eligibility is gated on active-paid status.

**Feedback & Bugs System — build-support infrastructure, not just a feature.** This exists so Taylor can talk to Claude about the app while it's being built and maintained, instead of re-explaining notes in chat every time. Query the `FeedbackItem` table (Supabase) directly at the start of a build or maintenance session to pick up any open bugs, feature ideas, or notes she's logged — don't wait for her to paste them into chat.

**Non-negotiable quality bar**
- One meal per employee per day enforced in UI and backend.
- Prices shown to employees are exact and final.
- Cutoff is impossible to miss.
- QR, self-confirm, proxy, and already-picked-up states are all clear.
- No-shows auto-mark and are visible.
- Money movement and credits are auditable.
- Admin users default to their normal daily experience.
- Daily open feels nearly instant via biometrics after first login.
- Only active employees count toward billing ceilings.
- Deactivation never silently destroys paid future orders.

**Definition of done for v1**
A caterer can log in, invite a company, publish composite meals for the next two weeks, and have real employees ordering, paying, picking up (including via proxy), and generating accurate headcounts and no-show data the same day.

---

## Appendix: Landing Page & Marketing Site Specification

**URL:** `lunchkit.app`

**1. Hero** — Caterer-focused headline + "Start free as a Caterer" + secondary Company/Employee paths
**2. Social Proof Bar**
**3. Problem Section** — Caterer / Company / Employee pains
**4. Solution Overview**
**5. How It Works** (4 steps)
**6. Audience Sections** (Caterer → Company → Employee)
**7. Key Features**
**8. Install / Get the App on Your Phone** (iPhone Safari steps, Android Chrome steps, desktop note)
**9. Pricing** (table + trial + annual + soft ceilings)
**10. FAQ**
**11. Final CTA**
**12. Footer** — Log in, privacy, terms, contact, `© 2026 Clique Studios IO. All rights reserved.`
**13. Find a Caterer (Phase 2)** — public directory page (`/caterers`), filterable by state/city, linked from the main nav and footer once the Caterer Directory feature ships. Not part of the initial marketing site build.

Additional rules: persistent Log in button, no dead ends on unknown email, calm premium scannable design, proper SEO.

---

**End of Product Plan**

This document is intentionally complete and unambiguous. It is `CLAUDE.md` at this repo's root — commit it there first, then begin implementation directly against the GitHub repo, following the build order and Git Workflow Rules above.
