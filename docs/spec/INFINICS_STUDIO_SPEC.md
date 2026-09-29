# Infinics Studio — product vision, target requirements and fit criteria

Version 1.0 · 30 September 2026 · Infinics Teknoloji A.Ş. · Working name: **Infinics Studio** (the name is a placeholder)

Status: the product exists as a clickable prototype (mock data, mock clicks) that has been used to settle the concept. This document turns that concept into requirements so that an open-source system can be checked for fit as a foundation.

---

## 0. How to use this document (instructions for Claude Code)

You are evaluating an open-source repository as a possible foundation for the product described below. Follow this procedure:

1. Read this whole document before opening the repository.
2. Map the repository: architecture, main concepts and their names, extension points (plugins, adapters, hooks), data model, UI layer, deployment model, license.
3. For every requirement (R-x.y) in sections 4–8 and every part (P-x) in section 5, rate the repository as one of: **Native** (exists as-is), **Adaptable** (exists, needs changes; estimate effort S / M / L: days / weeks / months for one senior engineer), **Missing** (must be built on top), **Conflicts** (the repository's design works against it and changing it would be a fork). Cite the files or modules that justify each rating. Verify in code; do not rate from the README.
4. Check the must-haves in section 9.1 first. A failed must-have that cannot be fixed with reasonable effort ends the evaluation with the verdict "not a fit"; say why and stop there.
5. Assess modifiability using section 9.3.
6. Write the report in the format of section 10. Be concrete and short; tables over prose. Where the repository uses a different word for one of our concepts, say so ("their *workflow* is our *solution*").

Two framing rules: the **user experience in sections 3–7 is the product**; the repository's UI can be replaced, its engine cannot be argued away. And the **user never sees technology**; if a capability only exists in the repository as a developer-facing feature, rate it Adaptable at best.

---

## 1. Vision

Business users at SMEs and mid-market companies should get the capability of an agentic coding assistant without any of its technical surface. A business user describes a task, a problem or a process in their own words to **the Analyst**, an AI role that clarifies requirements, writes a brief, proposes a blueprint, builds the solution, tests it against real past cases, launches it in stages and keeps improving it from what people actually do. The result is a working **agent**, an **application**, or both, running under the company's own workspace. Database, hosting, models, retrieval, integrations, deployment and technology selection never reach the user; they sit behind an optional technical view meant for IT and for Infinics as a partner.

Positioning that must hold (from Infinics' go-to-market): an agent connects to back-end systems (ERP, CRM, mail, WhatsApp, portals), consults documents and the web when needed, works with other solutions and **takes actions** in those systems; human-in-the-loop screens to manage and redirect agents matter as much as the agents; LLM judgment is non-deterministic while much of business needs determinism, so every solution blends **deterministic rules** (checked exactly, every time) with **judgment** (reading, drafting, spotting the unusual), and the blueprint says which is which.

## 2. Who uses it

| Role | Who | What they do | Sees the technical view? |
|---|---|---|---|
| Builder (owner) | A business user, e.g. Customer Operations Lead | Talks to the Analyst, approves blueprints, launches, sets limits, teaches rules | No (may switch it on) |
| Approver | Team member with limits | Approves cases within limits, teaches rules, asks for changes | No |
| Viewer / end user | Team members | Uses the generated screens, forms and chats through a team portal | No; never sees Studio |
| Outsider | Customers, suppliers | Uses public forms and customer chats at their own addresses | No |
| IT | The company's IT | Connects systems, approves write adapters, owns accounts and policies (data residency) | Yes |
| Partner | Infinics | Onboards workspaces, custom adapters, watches health across customers | Yes, without customer data |

## 3. Core concepts and vocabulary

| Term | Meaning | Suggested TR label |
|---|---|---|
| Workspace | One company (tenant): its solutions, connections, people, memory, addresses | Çalışma alanı |
| The Analyst | The single persona users build with. Asks, proposes, builds, tests, operates; may delegate internally to sub-agents (builder, tester) that stay invisible | Analist |
| Solution | One built thing with a lifecycle: Draft → Brief → Blueprint → Building → Tested → Live (shadow) → Live → Paused | Çözüm |
| Brief | Structured requirement summary that fills live during the conversation; editable | Gereksinim özeti |
| Blueprint | The contract before building: parts, flow, rules, when it asks a person, connections, guardrails, cost, addresses; versioned | Çözüm planı |
| Part | A building block the Analyst composes solutions from (section 5). The user never picks parts | Parça |
| Address | The URL (and channel) of a user-facing part: a screen, form or chat under the workspace domain | Adres |
| Apps & forms | The workspace directory of all addresses; the end-user portal lists what a person can open | Uygulamalar ve formlar |
| Connection | A system the workspace can use through an adapter (IT- or Infinics-provided), with plain-word permissions; read-only by default; writes are explicit adapters with quotas | Bağlantı |
| Analyst memory | Durable facts about the company and each person's preferences, reused across solutions, visible and editable | Analist hafızası |
| Suggested rule | A rule the Analyst proposes from what people actually did, with the cases as evidence; approving creates a tested version | Önerilen kural |
| Version | Every change to a solution (rule taught, change approved, suggestion accepted) is a version, tested before it goes live, with rollback | Sürüm |
| Under the hood | The optional technical view: models, workflows, adapters, data, deployment, health | Teknik görünüm |
| Shadow mode | The solution runs on real traffic but sends and writes nothing; its decisions are compared with the team's | Gölge mod |

## 4. The journey: six stages, same for every kind of request

### R-1 Describe (conversation and brief)
- R-1.1 Start from a free-text description; example starting points are suggestions, not templates the user must pick.
- R-1.2 The Analyst asks **one question at a time**, each with selectable answers, free text, and "Decide for me" (the Analyst picks a sensible default and marks it as a default).
- R-1.3 A structured **brief** fills live beside the conversation; a completeness measure and the open questions are visible; the brief is **editable in place** and the Analyst reacts to an edit by updating everything that depends on it (rules, approval step, cost).
- R-1.4 Workspace awareness: the Analyst already knows the connections, existing solutions and memory; it never asks what it knows, marks answers taken "from memory", and offers them as defaults.
- R-1.5 Documents, links and folders can be given during the conversation (PDF, DOCX, XLSX, a web page, a SharePoint/Drive folder); the Analyst says in plain words what it understood (rules found, sheets/columns/formulas found, form fields found) and what it will do with it.
- R-1.6 Business language only; technical terms appear only in the technical view.
- R-1.7 The conversation is persisted per solution and resumable across devices; later change requests continue the same thread and produce versions.
- R-1.8 Conversation in Turkish or English; customer-facing outputs in the customer's language.
- R-1.9 The Analyst names the parts it will use and why, in plain words ("there is no connection for that portal, so a browser worker"; "an Excel becomes records with the formulas as calculations"; "a write to Logo needs an adapter IT allowed").

### R-2 Blueprint (the contract)
- R-2.1 Generated from the brief: what you get (parts, named), how a request flows, rules split into **checked exactly** vs **handled with judgment**, when it asks a person split into **cannot be skipped** (enforced by the workflow) vs **when unsure** (the assistant's judgment), connections with a reason each and their status, guardrails, what is kept and for how long, estimated monthly cost, time to build, addresses.
- R-2.2 Nothing is built before approval. Change requests through the conversation produce a new blueprint version with a readable diff.
- R-2.3 "What done means": 3–5 testable acceptance criteria with an owner who accepts.
- R-2.4 Missing connections can be connected from the blueprint or during the build.
- R-2.5 A pending change that asks for more access (a new write, a new system) is flagged as such before approval.
- R-2.6 Technical view: the translation of the blueprint into models, flows, rules engine, knowledge index, data, adapters, approvals, safety, observability, deployment, cost basis.

### R-3 Build
- R-3.1 Automated build from the approved blueprint into the parts runtime (generated configuration, flows, screens, forms, records, indexes, schedules); progress shown in business language.
- R-3.2 The build pauses for the user when needed (connect a system, upload a file, IT account request) and resumes; the user is notified when it is done.
- R-3.3 Automatic tests with real past cases (with consent): past emails, invoices, applications, questions, board-report numbers; a report in business terms (decisions matched, sent to a person correctly, wrong); self-correction and re-test; a separate checker grades the fix; coverage is stated ("20 of 20 ran").
- R-3.4 Build logs and artifacts under the hood; everything versioned.

### R-4 Try (sandbox)
- R-4.1 One sandbox per part (section 5). Nothing real happens: sends are suppressed, writes go to test accounts.
- R-4.2 Agent traces in business language: what it read, found, checked (each rule ✓/✗), decided, drafted, and what happens next; the work steps fold into one line with "show details"; technical lines under the hood.
- R-4.3 Approvals are experienced in the sandbox: approve, reject, edit the reply; the card states whether the approval is enforced or asked.
- R-4.4 Feedback ("something's off") goes to the Analyst and changes the solution, re-tests it and creates a version, without leaving the screen.
- R-4.5 Generated screens can be changed through conversation ("add a courier column", "rename it") and are regenerated and re-tested safely.

### R-5 Launch
- R-5.1 Rollout modes: **shadow** (recommended for customer-facing solutions), **live with approvals**, **fully live**; later: fully live with a reviewer and a circuit breaker (Appendix A).
- R-5.2 People and roles per solution with approval limits; where approvals reach people (screen, email, Teams, WhatsApp); first answer wins.
- R-5.3 Limits: requests per day, spending cap per month, ratio alerts (e.g. refund rate), pause at any time.
- R-5.4 Addresses added by the solution, with who can open each; an inline confirmation stating in plain words what will and will not happen.

### R-6 Operate
- R-6.1 Outcomes first: handled automatically, asked a person, unclear, first-reply time, cost, "needed you N times"; a daily chart of automatic vs human.
- R-6.2 An attention queue mixing approvals, unsure cases, content gaps, stale data, readiness checks, with actions.
- R-6.3 Teach a rule → tested version; ask for a change → draft version → blueprint review → tested → live.
- R-6.4 **Suggested rules, content, definitions and reports** from patterns in what people did (overrides, shadow mismatches, repeated unanswered questions, repeated clarifications, repeated questions), each with evidence; approve or reject.
- R-6.5 Shadow-mode readiness: a comparison of the assistant's decisions with the team's, and a one-click move to the next mode.
- R-6.6 Versions timeline with rollback; an activity log where every action shows how it was decided (automatic, approved by whom, held by which rule, blocked).
- R-6.7 Technical view: uptime, latency, spend breakdown, connector errors, evaluation runs, traces.

### R-7 Workspace level
- R-7.1 Home: the describe box, a "waiting for you" inbox across solutions, the solutions list with business statuses, this week's totals.
- R-7.2 Apps & forms directory: every address with type (screen, form, chat), access (anyone with the link / team sign-in / specific people / role-based), solution and status; an end-user portal that lists only what the signed-in person can open; custom domain per workspace.
- R-7.3 Connections: business names (Logo, Netsis, SAP, Trendyol, Hepsiburada, WhatsApp Business, Google Workspace, SharePoint, Teams…), plain-word permissions per action (read / asks first / never), read-only by default, write adapters explicit with quotas, test vs real accounts, fields hidden from assistants, "used by N solutions", custom systems via IT or Infinics.
- R-7.4 Analyst memory: company facts (who approves, which systems, house rules, tone) and personal preferences; source, date and used-by per item; forget, correct, add; facts settled in conversations are added and marked; the user's explicit instructions outrank memory; no customer data, no credentials.
- R-7.5 Team: roles (owner, approver, viewer, partner), approval and spending limits, audit trail, SSO (Google Workspace / Microsoft).

### R-8 Cross-cutting
- R-8.1 Multi-tenant workspaces with strict isolation; deployable on Kubernetes/Docker in a Türkiye data centre or on-prem; data residency and KVKK controls; no mandatory vendor cloud.
- R-8.2 Internationalisation: UI in Turkish and English; generated apps in the workspace language; customer replies in the customer's language; Turkish text handled correctly everywhere (case, characters, number and currency formats).
- R-8.3 Observability and evaluations per solution; cost accounting per solution with caps.
- R-8.4 Security: PII masking before model calls, prompt-injection guard on inbound text, isolated execution for workers, credentials in a vault injected per run, RBAC and row-level filtering enforced outside the prompt, full audit.
- R-8.5 Extensibility: new parts, new adapters (MCP or equivalent), new models through a gateway, accelerator templates from the Infinics catalog (e.g. Invoice Intake, Quote Builder) as starting points.

## 5. Parts model

A solution is a composition of a fixed set of parts. The Analyst picks and names them; each user-facing part gets its own address. Grouped as the product shows them:

**Where people work**
- **P-1 Screen** — a place where the team works: queues, lists, numbers, case detail; team sign-in; own address; optional **helper** panel scoped to the screen (explains a decision, filters, drafts; cannot change decisions).
- **P-2 Form** — a page where people enter information and upload files; public (anyone with the link, bot check, TR/EN) or team; fields inferred from a description or an existing Word/Excel form; after-submit steps run by an assistant.
- **P-3 Chat** — a conversation window on a screen, in Teams, embedded on the website or on WhatsApp; every answer shows its source; refuses what the audience may not see (enforced by the library or the role, not by the model); hands over to a person with the conversation; asks for a definition instead of guessing a number.
- **P-4 Report** — a saved answer that keeps itself up to date (table or chart), sent on a schedule or shown on a screen; skipped with a note when data is stale.

**What it knows**
- **P-5 Records** — a shared table replacing a spreadsheet: columns with rules and choices, calculations that are never typed, history of every change with who and when, roles (edit/view), import from Excel once (formulas → calculations, messy columns → choices, corrections listed), export any time, scheduled export.
- **P-6 Library** — documents, policies and pages the assistant answers from, kept in sync with their source (SharePoint/Drive folders, web pages, uploads); collections with visibility per audience; duplicate handling; changed documents re-read within an hour with "which answers changed"; unanswered questions collected per collection as gaps; one library per workspace, shared by solutions.
- **P-7 Reader** — reads documents and photos (invoices, receipts, IDs, contracts): extracts fields and lines with a confidence per field; anything under a threshold goes to a review queue; matching against system records (e.g. purchase orders) with tolerances; duplicates caught; originals kept.

**What it does**
- **P-8 Assistant** — works on its own: reads, looks things up, applies rules exactly, decides, drafts, acts through adapters, asks a person when unsure; every action traceable in business language.
- **P-9 Browser worker** — performs steps on a website that has no connection, the way a person would: IT-owned account with minimal rights, allowlist of one site, every run recorded as step screenshots, stops and asks when the site changes, never enters payment data, one action per business object (no double filing); optional click-before-acting period.
- **P-10 Schedule** — runs something at set times; each run is a conversation that can be opened and asked a follow-up.

**How it is controlled**
- **P-11 Approval** — a human decision step with limits per person, reminders, timeouts, escalation and a record of who decided; enforced by the workflow, not by the model; the assistant can propose rules but never applies one.

Semantics for numbers (used by Chat/Report over company data): a **definitions** layer (metrics written once in plain words, with formula, source tables, owner, approval by Finance, versions); only defined metrics are calculated; role-based visibility before the query runs; every answer carries "how I calculated it".

## 6. Governance and trust (applies everywhere)

- G-1 Deterministic before judgment: policy checks, thresholds, visibility, tolerances and duplicates are rules in a rules engine; the model reads, drafts and flags.
- G-2 Two kinds of asking a person: enforced (workflow interrupt, cannot be skipped) and judgment (the assistant is unsure). Both visible in the blueprint and on every approval card.
- G-3 Hard floors: anything whose authority outlives the conversation (rules, schedules, guardrails, approval limits, connections, access) is changed only by a person; the assistant proposes, never applies.
- G-4 Approvals have honest lifetimes: approve once; approve all like this today; make it a rule (revocable, visible in a standing-rules list).
- G-5 Shadow mode first for anything customer-facing; readiness is measured against the team's decisions.
- G-6 Every action is logged with how it was decided and by whom; nothing vanishes silently; "needed you N times" is a first-class number.
- G-7 Memory: a wrong memory costs more than a missing one; sensitive facts need a yes; the user's own instructions outrank memory; forgetting is one click.
- G-8 Read-only by default; writes only through explicit adapters IT allowed, with quotas, naming the account they write to; test accounts for sandboxes.
- G-9 Workers run isolated: brain outside, hands inside; secrets injected per run and removed; fail closed if isolation cannot be verified; a refusal names the host.
- G-10 Data stays in the region (Türkiye); personal data masked before models; no personal data in customer chats; retention per solution.

## 7. Reference scenarios (use these as test cases for fit)

Each scenario names the parts the Analyst should compose, the connections it uses, and behaviours the foundation must be able to express. Mock company: Karya Home, a home-textiles brand selling on its own website plus Trendyol and Hepsiburada; ERP is Logo.

| # | Scenario | Parts | Connections | Behaviours to verify |
|---|---|---|---|---|
| S-1 | **Return Request Handler**: 40–60 return requests a day by email and WhatsApp; find the order in Logo; apply a 14-day policy from a document plus an unwritten rule (campaign items exchange only); refunds above a limit approved by the owner; a Returns Desk screen for 4 people | Assistant, Screen, Approval, Records | Gmail/inbox, WhatsApp Business, Logo (read), policy document | One question at a time with defaults; brief editable (change the limit); policy read from a PDF/URL; rules exact vs judgment; holding reply while waiting for approval; test on 20 past emails; shadow mode; teach "damaged on arrival"; suggested goodwill rule with evidence |
| S-2 | **Supplier Invoice Intake**: ~400 invoices a month as e-invoice PDFs and phone photos; extract fields and lines with confidence; review under 95%; match POs with 2% tolerance; post drafts to Logo through an IT-approved adapter; Accounting review screen with a helper | Reader, Assistant, Screen (+helper), Records, Approval | Mail, WhatsApp, Logo (read POs; write drafts via adapter, quota) | Per-field confidence and review queue; duplicate detection; write adapter with daily limit; helper explains/filters/drafts; suggested rule "accept rounding differences under 5 TL from supplier X" |
| S-3 | **Price List from Excel**: a 312-row Excel edited by 6 people becomes records with two formulas as calculations, a messy unit column as a choice, roles (Purchasing edits, Sales views), history, a form for new items, a Friday Excel export; another solution (Quote Assistant) reads it live | Records, Form, Screen, Assistant, Schedule | Google Drive/SharePoint (read once, write export) | Import report (14 corrected rows, 7 spellings → 3), calculations never typed, rule "campaign price cannot go below cost" stops a change and asks, live reuse by another solution |
| S-4 | **Supplier Onboarding**: a public application form built from an existing Word form, with document uploads; a checker validates the tax number in a registry, chases a missing document once, holds incomplete applications back; Purchasing approves; a supplier card is created in Logo through an allowed adapter | Form (public), Assistant, Records, Screen, Approval | Tax registry lookup, email, Logo (write supplier card via adapter) | Public address without sign-in, TR/EN, file limits, replay of last 15 applications, approval before any write |
| S-5 | **Courier Claims Filer**: damage/loss claims filed on a courier's corporate portal that has no API, with an IT-owned account; claims come from S-1 (damaged items) and a team form; click-before-filing for the first month; run recordings; the worker stops when the portal changes | Browser worker, Assistant, Form, Records, Screen, Approval | Portal (browser), Logo shipments, S-1 hand-off | Step recordings, single-site allowlist, paused-run handling and fix, "let it file on its own" suggestion after 44/44 unchanged |
| S-6 | **Company Knowledge Assistant**: one library synced from SharePoint and the website (214 documents, 38 pages) in collections with visibility (Products and Returns & shipping for everyone; HR and Procedures employees-only); an employee chat (portal, Teams) and a customer chat (website widget, WhatsApp); every answer cites its source; no source → hand-over and a collected gap; prices only from S-3 records; tested on 120 past support questions | Library, Chat ×2, Assistant, Approval | SharePoint, website, Teams, WhatsApp, S-3 records | Visibility enforced by the index, not the prompt; citations; hand-over with conversation; changed-document propagation; suggested content from repeated gaps; shadow week for the customer chat |
| S-7 | **Company Insights**: a chat over hourly read-only reporting copies of Logo and the web database; metric definitions written with Finance (net sales, return rate, gross margin, stock days) and approved before use; role-based visibility (C-level all, Sales no margins, Ops stock/returns, no personal data); every answer shows "how I calculated it"; undefined words are asked about; any answer can become a scheduled report; checked against the board report before launch | Chat, Assistant, Records (definitions), Report, Schedule, Approval | Reporting replicas, Teams, email | Semantic layer instead of free SQL; RLS before the query; clarification instead of guessing; suggested definition from repeated clarifications; suggested report from repeated questions; stale-data skip |

## 8. Target architecture and technical principles

Preferred stack (a candidate may replace components; rate the replacement cost):
- Agent flows: LangGraph; durable workflows with human interrupts and schedules: Temporal (or equivalent); deterministic rules engine for policy checks.
- Models: LiteLLM as a model-agnostic gateway (Anthropic, Azure OpenAI, Gemini, Bedrock, plus self-hosted open weights via vLLM on GPUs in Türkiye, incl. vision models for photos).
- Integrations: MCP for ERP/CRM/mail/WhatsApp/portals; write adapters with quotas; connector permission scoping.
- Knowledge and data: PostgreSQL (records with history, row-level security) + Qdrant (hybrid retrieval, one collection per visibility); object storage in-region for originals and recordings.
- Safety: Presidio (PII masking), LLM Guard (prompt injection); vault for credentials; isolated browser pods (Playwright) for workers.
- Observability: Langfuse traces and evaluations; per-solution cost accounting.
- Front end: Next.js/React for Studio and generated apps; Teams app, website widget, WhatsApp Cloud API as channels.
- Packaging: Docker/Kubernetes; Türkiye data centre, on-prem or customer cloud.

Principles:
- A-1 **Generated UI is declarative, not free-form**: parts are a controlled vocabulary; screens and forms inside parts are composed from a typed component catalog (tables, forms, tiles, charts, queues, detail views), so "add a column" is a schema change, not code generation, and every generated app looks like one product. Free-form generated UI is allowed only for one-off visualisations in the sandbox.
- A-2 **A protocol between UI and agents** with streaming state, generative UI, interrupts/approvals and channel adapters (AG-UI is the current candidate) rather than bespoke sockets.
- A-3 **Parts expose tools**: screens, forms and records expose their actions so solutions compose through tools (S-1 creates a claim through S-5's tool), never through shared tables.
- A-4 **Definitions before answers** for numbers: a semantic layer of approved metrics; the model selects metrics, dimensions, filters and periods and never writes free SQL.
- A-5 **Visibility and roles are enforced below the model**: collection ACLs at query time, RLS in the database session, adapter permissions in the connector.
- A-6 **Evaluations are part of the build**: every solution ships with a test set from real past cases and is re-run on every version and every document sync.
- A-7 **Everything is a version**: blueprint, generated configuration, rules, definitions, test sets; rollback is one step.

## 9. Fit criteria for an open-source candidate

### 9.1 Must-haves (fail one → not a fit unless fixable with S/M effort)
- M-1 License allows commercial SaaS use and modification (MIT, Apache-2.0, BSD preferred). AGPL, BSL, SSPL or "open core with essential features closed" require a separate decision; state which it is.
- M-2 Self-hostable on Kubernetes/Docker in a Türkiye data centre with no mandatory vendor cloud for core features (auth, storage, models, integrations, UI).
- M-3 Multi-tenancy or a clean path to per-workspace isolation (data, credentials, addresses, models, cost).
- M-4 Model-agnostic: works through a gateway and with self-hosted open-weight models; no hard dependency on one provider.
- M-5 Durable workflows with human-in-the-loop interrupts, timeouts and schedules (native, or integrable with Temporal/LangGraph without fighting the framework).
- M-6 An extensible integration layer (MCP or an adapter model) with per-connection permission scoping and a way to distinguish reads from writes.
- M-7 A UI we can theme, extend or replace: either a component-based web UI with slots, or headless APIs/events that let us build the UX of sections 3–7. Non-negotiable: the business-user experience described here.
- M-8 Active maintenance: commits within the last 3 months, issues answered, a test suite, docs, a release process.
- M-9 Observability that we can route to Langfuse (or equivalent) and per-run cost data.
- M-10 Unicode-safe and i18n-ready (Turkish text, number and date formats); no English-only assumptions in prompts that we cannot change.

### 9.2 Weighted areas (score each 0–3: 0 missing/conflicts, 1 weak, 2 adaptable, 3 native)
| Area | Weight | What to look at |
|---|---|---|
| Conversation-to-spec (R-1, R-2) | 15 | Structured requirement capture, editable state shared between user and agent, versioned specs |
| Build and test (R-3, A-6) | 15 | Generating configuration/flows/screens from a spec; evaluation harness on real data; versioning |
| Parts coverage (P-1 … P-11) | 20 | Which parts exist natively; how a new part type is added |
| Governance (G-1 … G-10, R-5) | 15 | Interrupts, approval limits, rollout modes, audit, memory controls |
| Operate and learning (R-6) | 10 | Outcomes, attention queue, suggestions from evidence, versions/rollback |
| Workspace features (R-7) | 10 | Directory of addresses, portal, connections with permissions, memory, roles |
| Architecture and security (R-8, A-1 … A-7) | 15 | Multi-tenancy, residency, masking, isolation, RLS, semantic layer, declarative UI |

### 9.3 Modifiability
- Architecture: modular? plugin/extension system? clear boundaries between engine, integrations, storage and UI?
- Code quality and tests: coverage, typing, CI; how risky is a change in the core?
- Docs: architecture docs, extension guides, upgrade notes.
- Community and vendor risk: bus factor, company behind it, roadmap transparency, breaking-change history.
- Upstream strategy: could we build on top as a plugin/app and keep pulling upstream, or would we fork? Estimate the size of the fork in modules.
- Data model fit: can our concepts (solution, blueprint, part, address, version, memory) be represented without hacks?
- Deployment fit: images, Helm charts, resource needs, GPU support for self-hosted models.

## 10. Report format (what Claude Code should produce)

1. **Verdict** (one of): Adopt as foundation / Adapt (with the list of changes) / Take pieces (which modules, for which parts) / Build ourselves. One paragraph of reasoning.
2. **Must-have check**: M-1 … M-10 with pass/fail and evidence (file paths).
3. **Area scores**: the table of 9.2 with score, weighted score, and one line of evidence per area.
4. **Requirement ratings**: a table for R-1 … R-8 (each sub-item) and P-1 … P-11: rating (Native / Adaptable S-M-L / Missing / Conflicts), evidence, notes.
5. **Top 10 gaps** ordered by weighted importance, each with effort and an integration idea.
6. **Risks**: license, lock-in, maintenance, security, performance, i18n.
7. **Integration strategy** if the verdict is Adopt or Adapt: what stays upstream, what we add, what we replace, how we keep upgrading.
8. **Vocabulary map**: their term → our term.

Keep the report under 2,500 words plus tables. Cite code, not marketing.

## 11. Roadmap (proposed sequencing; adjust as needed)

- **Phase 0 (done):** clickable prototype; concept, journey, parts model, seven scenarios, governance ideas validated.
- **Phase 1 (MVP):** workspace, Home, Describe with live brief, Blueprint with approval, Build with tests on past cases, Try (agent trace, screen preview), Launch (shadow, approvals), Operate (outcomes, attention, teach a rule, versions). Parts: Assistant, Screen, Records, Approval, Schedule. Connections: mail, WhatsApp Business, Logo read via adapter. Scenarios S-1 and S-3. Turkish UI. Technical view for IT.
- **Phase 2:** Form (public), Reader, Library + Chat, Report; Apps & forms directory and end-user portal; Analyst memory; suggested rules/content with evidence; write adapters with quotas. Scenarios S-2, S-4, S-6.
- **Phase 3:** Browser worker; Insights with a definitions layer; approvals in Teams/WhatsApp; reviewer and circuit breaker for fully-live mode; helper panels in screens; partner multi-workspace view; accelerator templates from the Infinics catalog. Scenarios S-5, S-7.

## 12. Open questions (not yet decided)

- Persona: one Analyst, or specialised roles the user can see? (Current answer: one face; delegation invisible.)
- Is the set of 11 parts right? Candidates: customer chat widget as its own part vs a Chat channel, mobile app, dashboards beyond Report.
- Who sees the technical view: owners on request, or IT/partner only?
- One shared Library per workspace (current) vs per solution.
- Custom domain per workspace vs a shared domain; embeddable public forms.
- Who may see run recordings and uploaded documents (solution users, IT, Infinics).
- Memory: apply facts learned in one solution silently in another, or confirm once; company facts vs personal preferences, who edits which.
- Suggested rules: minimum number of cases before a suggestion; expiry.
- Shadow mode mandatory for customer-facing solutions? Who can launch (owner only)?
- Who connects systems (business user, IT, partner); billing model (per solution, per task, per seat).

---

## Appendix A — Ideas adopted from other products (for later phases)

From **CopilotKit** (agent-in-app framework): the generative-UI spectrum (controlled parts, declarative screens, free-form only in sandboxes); bi-directional shared state (editable brief); two human-in-the-loop patterns (model-requested vs graph-enforced interrupt → our "when unsure" vs "cannot be skipped"); rich resumable threads; automatic learning from completed runs → our suggested rules with evidence; user/project memories with explicit save and forget; channels (one agent, native cards in Slack/Teams); an in-app copilot with app context and frontend actions → our helper panel; AG-UI as the wire protocol candidate.

From **OpenWorker** (local-first AI coworker app): approval card anatomy (verb, reason, scope line, preview, provenance); a grant ladder with honest lifetimes; hard floors for anything that outlives the conversation; a reviewer model that judges scope, defaults to "unsure", never unblocks, runs in shadow first, and a circuit breaker after repeated denials; falsifiable acceptance criteria and "a fixer never grades its own fix"; "declared intent, not an access grant" with per-connection reasons; connector controls in plain words (read / asks first / never, test vs real accounts, hidden fields); memory rules; an activity log where nothing vanishes silently; runs as conversations; sandbox principles (brain outside, hands inside; secrets absent not denied; fail closed).

## Appendix B — The prototype

A single-page clickable prototype ("Infinics Studio") shows every screen above with mock data: Concept (journey, parts, "what you say / what Studio does"), Home, Describe (interactive for S-1; completed transcripts for S-2…S-7), Blueprint, Build, Try (per-part sandboxes), Launch, Operate (suggested rules), Solutions, Apps & forms (with the end-user portal), Connections, Analyst memory, Team; a Notes panel lists what each screen demonstrates and the questions to discuss. The HTML file can be provided alongside this document for reference; treat it as illustration of the intended UX, not as code to reuse.
