# Rome as a foundation for Infinics Studio: fit report

Evaluated: [`rome-os/rome`](https://github.com/rome-os/rome) at commit `ef523c4` (v1.1.129, 30 September 2026), against [`../spec/INFINICS_STUDIO_SPEC.md`](../spec/INFINICS_STUDIO_SPEC.md) v1.0.
Method: six parallel code reviews (tenancy and auth, durable execution, integrations, UI and i18n, data and security, runtime and operations), then first-hand checks of every claim the verdict rests on. Paths are relative to the Rome repository root. Nothing was deployed. The only code run was a Node check of Rome's case folding on Turkish strings.

Ratings: **Native** exists as-is · **Adaptable S/M/L** exists and needs days / weeks / months of one senior engineer · **Missing** must be built · **Conflicts** the design works against it and changing it means a fork.

---

## 1. Verdict

**Take pieces.**

**What Rome is.** A carefully engineered and very active *personal* agent OS, built around three choices:
- One instance serves exactly one "guardian" (`docs/concepts/people.md`: "There is no multi-guardian mode").
- Its agents run inside Anthropic's Claude Agent SDK or OpenAI's Codex, with a shell and no permission prompts.
- Every new capability is free-form TypeScript and React written by a coding agent.

**Where Studio needs the opposite.** On the three axes that decide the product:
- A company workspace with several roles and per-person approval limits.
- A controlled vocabulary of parts, with declarative generated screens and forms.
- Governance the platform enforces rather than the model: read-only by default, shadow mode, isolated workers.

**Must-haves and score.** No must-have fails beyond a medium-sized fix. Three fail as shipped: M-2 (hosted dependencies), M-5 (durability) and M-6 (permission scoping). The weighted score is **100 / 300 (33%)**.

**Why not adapt it.** Adapting Rome would mean forking auth, approvals, the people model, connections, mail and OAuth, and the security model of a 250k-line TypeScript codebase (plus 170k lines of tests). That codebase landed 332 commits in its first 39 public days. We would still have to build every business-facing part ourselves.

**Recommendation.** Build on the preferred stack in section 8 of the spec. Take these of Rome's designs, plus a few leaf modules, under its MIT license:
- the interaction and suspension protocol
- the action result envelope and manifest packaging
- the channel port model
- the connection setup protocol
- the session and trace model
- the "spec before build" discipline

## 2. Must-have check

| Must-have | Result | Evidence | Fix |
|---|---|---|---|
| M-1 License | **Pass, two flags** | `LICENSE` is MIT, not AGPL, BSL or SSPL.<br>**Flag 1:** the default agent path imports `@anthropic-ai/claude-agent-sdk` (`packages/core/src/core/anthropic-provider.ts:1`). Its LICENSE.md reads "© Anthropic PBC. All rights reserved". The image also installs the Claude Code CLI (`Dockerfile:240`).<br>**Flag 2:** everything involving other parties runs on the closed control plane `amantru/rome-cloud` (`docs/architecture/observability.md`): provisioning, visitor sign-in, the OAuth broker, mail, the webhook relay and the store. That is open core for multi-party features, so it needs the separate decision. | — |
| M-2 Self-hosted, no vendor cloud | **Fail as shipped, fixable (M)** | Without Rome Cloud:<br>• Visitor sign-in returns 503 (`api/routes/visitor-auth.ts:196-199`).<br>• OAuth exists only for google, github and slack, through Rome Cloud (`lib/oauth-providers.ts:1`, `lib/rome-cloud-oauth.ts:72-168`).<br>• Email exists only through Rome Cloud "Rome Mail" (AgentMail) (`index.ts:992-1014`, `lib/rome-cloud-mail.ts:7-11`).<br><br>Gmail, Outlook, Drive and HubSpot go through hosted Composio (`rome_apps/connector/src/api/composio-client.ts:4`). The published image bakes in GA4 (`Dockerfile:175`), and the web build bakes in romeos.cc (`packages/web/rsbuild.config.ts:35-49`). The agent core itself runs locally. | M: our own identity provider and OAuth clients, IMAP or Microsoft Graph mail, MCP adapters in place of Composio, and our own images |
| M-3 Tenancy | **Pass (one instance per workspace)** | "Single-tenant by design" (`packages/core/src/api/index.ts:112-118`). One process per profile, with no tenant columns (`docs/architecture/process.md`). Data, credentials, origin, model settings and cost all live in the container. The provisioner is closed. Several users *inside* one workspace is a separate gap (R-7.5). | M: our own Kubernetes provisioner |
| M-4 Model-agnostic | **Conditional pass (M)** | Only two agent runtimes: `ProviderId = "anthropic" \| "openai" \| "mock"` (`core/agent-runner.ts:30`).<br>• A custom Anthropic-compatible endpoint is supported (`ANTHROPIC_BASE_URL`, `ANTHROPIC_MODEL`; `api-types/src/anthropic-compatible-env.ts`, `core/anthropic-provider.ts:382-513`), so LiteLLM in front of vLLM is plausible but untested.<br>• Codex gets only a fixed environment allowlist (`core/codex/common.ts:23-32`).<br>• Tier maps hard-code Claude and GPT model IDs (`core/model-resolver.ts:110-114,150-154`).<br>• Images reach only the Codex path (`core/codex/turn-input.ts:15-20`).<br>• The agent loop is always a vendor harness. | M: gateway configuration, vision and tier remapping. L for an agent loop of our own |
| M-5 Durable workflows with human-in-the-loop | **Fail as shipped, fixable (M)** | **What exists:** approval-gated actions persist and resume through a replay journal (`actions/engine.ts:558,1791-1808`, `actions/approval-handler.ts:164-213`).<br>**What is missing:**<br>• There is no timeout or escalation, and nothing auto-resolves (`actions/approval-flow.e2e.test.ts:504`).<br>• A crash marks the run as an error (`engine.ts:905-913`).<br>• Workflow apps throw on `pending_approval` (`packages/app-template/workflow/src/actions/run/index.ts:41-49`).<br>• Schedules are in-memory cron jobs recomputed from "now" at boot (`routines/engine.ts:45-77`), and the event bus is in memory (`events/event-bus.ts:20-25`). | M: Temporal alongside Rome, with our P-11 built on it |
| M-6 Integration layer | **Fail as shipped, fixable (M)** | **Grants:** there are named grants with talk, act and watch capabilities (`connections/types.ts:19`). There is no read-only grant and no quota, and each service allows one connection (`db/schema/system.ts:862`).<br>**Action flags:** `sideEffects` is metadata that nothing enforces (`app-runtime-sdk/src/index.ts:133`). `connector_proxy` exposes the provider's whole API (`rome_apps/connector/app.yaml`).<br>**Extension:** new adapters can only be added in core (`connections/registry.ts:193`). MCP servers declared in an app (stdio only, `apps/packaging/artifact-config.ts:102-111`) reach the agent only when network discovery is on (`core/agent-session.ts:998-1003`). | M: an MCP gateway with a permission policy and quotas, outside Rome |
| M-7 Replaceable UI | **Pass (M)** | A headless REST and SSE API (`api/routes/webchat.ts:186-195,2703`) with a typed but private contract (`packages/api-types`). App bundles mount through a small Shadow DOM contract (`packages/web/src/components/rome-app-host.tsx`, `packages/app-web-sdk/src/cli/generateEntry.ts:28-73`). Auth is a same-origin cookie, so a separate frontend needs a backend-for-frontend. | M |
| M-8 Maintenance | **Pass, maturity risk** | **Strengths:**<br>• 332 commits since the 22 August 2026 code drop, nearly all squash-merged PRs (up to #555).<br>• 810 test files, and CI runs typecheck, lint, unit, integration and e2e (`.github/workflows/ci.yml`).<br>• release-please with 31 `v*` tags, extensive docs and 24 ADRs.<br><br>**Risks:**<br>• Public for only 39 days, and one author wrote 54% of the commits.<br>• The issue tracker is private (`amantru/rome-internal`), and there is no SECURITY.md.<br>• The coverage gate runs only nightly (`packages/core/rstest.config.ts:54-58`).<br>• How quickly issues get answers was not checked. | — |
| M-9 Observability | **Pass (S)** | OTLP traces, metrics and logs (`telemetry.ts:43-111`) go to a collector whose exporter is configuration (`infra/observability/otel-collector-config.yaml:59-71`), so Langfuse is a config change. Tokens and `cost_usd` are stored per turn with session, turn and trigger IDs (`db/schema/system.ts:683-703`), and a metric breaks them down by agent and app (`telemetry.ts:318-322`). There are no `gen_ai.*` attributes and no span per model call. | S |
| M-10 Unicode and i18n | **Pass (M)** | **UI:** i18next ships only en and zh-CN, and every non-Chinese browser falls back to en (`packages/web/src/i18n/index.ts:53-56`). Currency is fixed to USD (`pages/sessions-format.ts:20-24`).<br>**Text handling:** case folding fails on Turkish (`api-types/src/people.ts:173-185`). "istanbul" does not match "İstanbul", and "ışık" does not match "IŞIK" (checked in Node).<br>**Prompts:** they are in English, with no reply-language setting (`app-runtime-sdk/src/conversation-settings.ts`). But app agents own their whole prompt (`core/prompt-builder.ts:300-308`), so nothing here is locked. | M |

## 3. Area scores

| Area (weight) | Score | Weighted | Evidence |
|---|---|---|---|
| Conversation-to-spec (15) | 1 | 15 | Question cards offer single-choice, multi-choice or text answers plus free text, with no default (`core/mcp-facade.ts:1071-1119`). The spec is prose, signed off in chat (`rome_apps/workflow-studio/src/skills/design-workflow/SKILL.md`). There is no brief object, no versions and no diff. |
| Build and test (15) | 1 | 15 | A coding agent scaffolds, builds and installs an app from the approved spec (`rome_apps/coding/src/skills/workflow_creation/SKILL.md`). The output is free-form code, and nothing tests it against real past cases. |
| Parts coverage (20) | 1 | 20 | Assistant, Schedule and Report can be adapted. Screen, Form and Chat need a declarative layer (L). Records, Library and Reader are missing. Browser worker and Approval conflict. A new part type would be a new app, which works well. |
| Governance (15) | 1 | 15 | An approval flag enforced in code exists, but only a demo action uses it (`rome_apps/system/src/actions/request-approval-demo`). There is no rules engine, rollout mode or limit. Memory and skill writers that act on their own, and agents with a shell, conflict with G-3, G-8 and G-9. |
| Operate and learning (10) | 1 | 10 | There is an execution log, an approvals page, traces and cost. Learning agents apply changes directly instead of proposing them. There are no outcome figures, no shadow readiness and no one-step rollback. |
| Workspace features (10) | 1 | 10 | An Apps page with per-app access, connection pages and a memory browser. One guardian only, with no roles, no SSO and no end-user portal. |
| Architecture and security (15) | 1 | 15 | One container per tenant and actions as tools are strong. UI is free-form, there is no RLS, semantic layer or PII masking, and all code inside the container is trusted. |
| **Total (100)** | | **100 / 300 (33%)** | |

## 4. Requirement ratings

**R-1 Describe**

| Req | Rating | Evidence and notes |
|---|---|---|
| R-1.1 Free-text start | Native | Webchat sessions (`api/routes/webchat.ts:2014`). The welcome app's idea cards are suggestions, not templates. |
| R-1.2 One question, options, "Decide for me" | Adaptable S | `ask_question` supports single, multi and text answers plus free text (`core/mcp-facade.ts:1071-1119`). A card can hold several questions. There is no default or "Decide for me" field. |
| R-1.3 Live editable brief | Missing | The spec lives only as prose in chat (`design-workflow/SKILL.md`). There is no shared state object. |
| R-1.4 Workspace awareness | Adaptable M | The prompt carries installed apps, `MEMORY.md` and project summaries (`core/prompt-builder.ts:335-535`). Connection status is "not known here" (`rome_apps/connector/app.yaml`), and nothing is marked "from memory". |
| R-1.5 Documents, links and folders | Missing | No PDF, DOCX or XLSX parsing and no OCR. Only the WebFetch tool reads web pages. |
| R-1.6 Business language only | Adaptable M | There is a voice guide (`docs/ui/VOICE.md`) and a rule to narrate in plain sentences, but traces show raw tool calls. |
| R-1.7 Persisted, resumable thread | Adaptable M | Sessions are durable and never expire (`docs/concepts/sessions.md`). They are not tied to a solution, and changes create no versions. |
| R-1.8 Turkish and English, customer's language | Adaptable M | There is no reply-language control (`app-runtime-sdk/src/conversation-settings.ts:3-11`), and prompts are in English. |
| R-1.9 Names parts and why | Missing | There is no parts vocabulary. |

**R-2 Blueprint**

| Req | Rating | Evidence and notes |
|---|---|---|
| R-2.1 Full blueprint | Missing | The spec covers trigger, steps, decisions, per-item work and delivery only, and it is not stored. |
| R-2.2 Build only after approval; versions and diff | Adaptable M | "Build only after a clear yes" is a prompt rule the model judges, not a code gate. There are no versions or diff. |
| R-2.3 Acceptance criteria | Missing | — |
| R-2.4 Connect from blueprint or build | Adaptable S | "Build first, connect after", using the `connect-card` inline component (`connector/src/actions/connector-connect/index.ts:118-124`). |
| R-2.5 Flag access increases | Missing | — |
| R-2.6 Technical translation | Missing | The app source is the only technical view. |

**R-3 Build**

| Req | Rating | Evidence and notes |
|---|---|---|
| R-3.1 Automated build into the parts runtime | Adaptable L | The build exists but emits code (`coding/src/skills/app_creation/REFERENCE.md:845`: the web entry "must default-export a React component"). It would have to be retargeted to emit configuration (A-1). |
| R-3.2 Pause, resume and notify | Adaptable M | Inline interactions resume the turn, and notifications exist. |
| R-3.3 Tests on real past cases | Missing | Only golden-trace tests against a mock model (`test/golden-trace-harness.ts`) and 38 AI-driven UI tests. |
| R-3.4 Logs and artifacts, versioned | Adaptable S | Each app is its own git repo plus a packed artifact, and turn traces are stored. |

**R-4 Try**

| Req | Rating | Evidence and notes |
|---|---|---|
| R-4.1 Sandbox with sends suppressed | Missing | `ctx.dryRun` is a convention the generated code must follow, and `runAction` ignores it (`app-template/workflow/src/actions/run/index.ts`). With one connection per service there are no test accounts. |
| R-4.2 Business-language traces | Adaptable M | Trace blocks (`api-types/src/trace-segments.ts:29-197`) are technical, with no ✓/✗ per rule. |
| R-4.3 Approvals in sandbox, enforced vs asked | Adaptable M | Approval cards exist. There is no "edit the reply" option, and the card does not say whether the approval is enforced or asked. |
| R-4.4 Feedback creates a version | Missing | The feedback route sends product feedback to Rome Cloud. |
| R-4.5 Change screens by conversation | Adaptable M | The coding agent edits and reinstalls the app. That is a code edit, not a schema change, and nothing re-tests it. |

**R-5 Launch**

| Req | Rating | Evidence and notes |
|---|---|---|
| R-5.1 Rollout modes | Missing | — |
| R-5.2 People, limits, approval channels | Conflicts | Approvals are guardian-only (`api/routes/approvals.ts:19`). Telegram and email receive only text (`channels/telegram.ts:222`, `channels/email.ts:170-177`). |
| R-5.3 Limits and pause | Missing | Apps and routines can only be disabled. There are no caps or ratio alerts. |
| R-5.4 Addresses with access | Adaptable M | Each app is private, public or email-restricted (`lib/public-access-config.ts:3-13`). Visitor sign-in needs Rome Cloud. |

**R-6 Operate**

| Req | Rating | Evidence and notes |
|---|---|---|
| R-6.1 Outcomes first | Missing | Only technical session and cost rollups (`db/repositories/webchat.ts:2057-2140`). |
| R-6.2 Attention queue | Adaptable M | The Activity page lists approvals and executions (`packages/web/src/pages/ActivityPage.tsx`). |
| R-6.3 Teach a rule, get a tested version | Missing | — |
| R-6.4 Suggestions with evidence | Adaptable M | The dream and skill-review agents mine conversations but *apply* the changes to memory and skills (`rome_apps/dream/src/agents/dream.yaml:99`, `skill-review.yaml:4`, both declared `acceptEdits`). They would need a proposal step. |
| R-6.5 Shadow readiness | Missing | — |
| R-6.6 Versions, rollback and activity log | Adaptable M | Installs are content-addressed, and one previous version is kept (`apps/manager.ts:40`). There is no rollback operation. The execution actor is overwritten on approval replay (`actions/engine.ts:1550-1553`). |
| R-6.7 Technical view | Adaptable S | Traces, cost, OTEL and uptime exist; evaluation runs do not. |

**R-7 Workspace**

| Req | Rating | Evidence and notes |
|---|---|---|
| R-7.1 Home | Adaptable M | Chat home, a widget grid (`packages/web/src/pages/free/`) and Activity. No business statuses. |
| R-7.2 Apps & forms directory and portal | Adaptable M | Apps page and standalone `/full/apps/<id>` (`packages/web/src/App.tsx:127-134`). No portal, roles or custom domain (single origin, `lib/caddyfile-generator.ts:280-284`). |
| R-7.3 Connections with permissions | Adaptable L | Grants and a generic setup protocol (`docs/architecture/channels.md`). None of the permission levels, quotas, test/real split or hidden fields. |
| R-7.4 Analyst memory | Adaptable M | Markdown with git history and an editor (`api/routes/memory-files.ts`). No per-fact source or used-by. By default it stores profiles of other people, which conflicts with "no customer data". |
| R-7.5 Team, roles, SSO | Conflicts | One seat (`api/routes/onboard.ts:94-98`, `db/schema/system.ts:418-436`). Dashboard visitors are flat co-admins (`api/routes/auth.ts:409-418`). No SSO. |

**R-8 Cross-cutting**

| Req | Rating | Evidence and notes |
|---|---|---|
| R-8.1 Isolation, residency, Kubernetes | Adaptable M | One container per workspace. Docker Compose only, with no Helm chart. Hosted defaults would have to be removed (M-2). |
| R-8.2 Turkish handling | Adaptable M | See M-10. |
| R-8.3 Observability, evals and cost caps | Adaptable M | OTEL and cost are recorded per session and app. There are no evaluations and no caps. |
| R-8.4 Security | Missing | No PII masking. Inbound triage is an LLM (sentinel) that fails open. No sandbox. Credentials are stored as plain JSON (`connections/ledger-db.ts:5-8`). No RLS. |
| R-8.5 Extensibility | Adaptable M | Apps are a real plug-in model. MCP is weak (see M-6). Models can go through a gateway via the Anthropic-compatible endpoint. The store backend is closed, but local installs work. |

**P-1 … P-11 Parts**

| Part | Rating | Evidence and notes |
|---|---|---|
| P-1 Screen | Adaptable L | App UIs are free-form React with a standalone route. There is no typed component catalog and no helper panel. `@rome-os/ui` has no data grid, chart or queue (`packages/ui/README.md`: "filling up"). |
| P-2 Form | Adaptable L | An app UI with public-link access. There is no form engine, bot check or import from a Word or Excel form. |
| P-3 Chat | Adaptable L | Webchat is guardian-only. Channels: Telegram, Discord, Feishu, WeChat, WhatsApp through the unofficial Baileys client (`channels/whatsapp.ts:1-13`), and email through Rome Mail. There is no Teams, no website widget, no citations and no audience enforcement. |
| P-4 Report | Adaptable M | Workflow apps, routines and the briefing app deliver on a schedule. There is no saved answer that keeps itself up to date, and no skip-when-stale. |
| P-5 Records | Missing | App tables are `<prefix>__*` in one shared SQLite file (`apps/context.ts:84-92`). There is no history, roles, calculations or import. |
| P-6 Library | Missing | No embeddings or retrieval. The only sync source is git (`sync/registry.ts:17-19`). |
| P-7 Reader | Missing | No extraction or OCR. |
| P-8 Assistant | Adaptable M | Agents, actions, `system:summon`, subagents and traces. Governance is weak. |
| P-9 Browser worker | Conflicts | A single persistent Chrome profile holds the guardian's logins (`scripts/docker/rome-start-chrome-cdp.sh:10-15`). "Rome never holds a LinkedIn credential; it holds the right to use the logged-in browser" (`channels/linkedin-cli.ts:5-6`). There is no allowlist, recording or per-run credential. |
| P-10 Schedule | Adaptable M | Schedule, event and manual triggers (`packages/core/src/index.ts:598-612`). Missed fires are skipped, and there is no follow-up on a run. |
| P-11 Approval | Conflicts | Guardian-only, and a fixed yes/no per action that ignores arguments. There are no limits, reminders, timeouts or escalation. |

**Governance and principles (compact)**

| Item | Rating | Note |
|---|---|---|
| G-1 Rules before judgment | Missing | The policy engine routes inbound messages only (`core/policy-engine.ts:47-135`). |
| G-2 Enforced vs judgment asks | Adaptable M | `requiresApproval` is enforced by the engine (`actions/engine.ts:558`); `ask_question` is the model's judgment. |
| G-3 Hard floors | Conflicts | Memory and skill writers act on their own. `create_routine` goes live "with no confirmation card". Callers inside the container on loopback count as the guardian (`lib/guardian-session.ts:28-49`). |
| G-4 Approval lifetimes | Missing | — |
| G-5 Shadow first | Missing | — |
| G-6 Every action logged | Adaptable M | `action_executions.actor` and `initiator` (`db/schema/system.ts:616-641`). No "needed you N times". |
| G-7 Memory rules | Adaptable M | Privacy is enforced only by the prompt, and there is no "sensitive needs a yes". |
| G-8 Read-only by default | Conflicts | `connector_proxy` exposes the full API, and no grant is read-only. |
| G-9 Isolated workers | Conflicts | The main agent has Bash with `bypassPermissions` (`packages/core/agents/main.yaml:4-19`) in the container that holds the database. Compose adds `SYS_ADMIN` and `apparmor:unconfined` (`docker-compose.yml:53-61`). |
| G-10 Region and PII | Adaptable M | Self-hosting in Türkiye is possible once the hosted defaults are removed. There is no masking. |
| A-1 Declarative UI | Conflicts | Free-form generated code is Rome's app model (`coding/src/skills/app_creation/AUTHORING.md:75`). |
| A-2 UI↔agent protocol | Adaptable M | SSE turn stream plus `pending_interaction` parts. An AG-UI adapter would be needed. |
| A-3 Parts expose tools | Native | Actions are tools that agents and apps call across apps. One caveat: all apps share one database handle. |
| A-4 Definitions before answers | Missing | — |
| A-5 Visibility below the model | Missing | No RLS, and the agent can read `rome.db` from the shell. |
| A-6 Evals in the build | Missing | — |
| A-7 Everything is a version | Adaptable M | Git and content-addressed bundles. Blueprints, rules and test sets are not versioned. |

## 5. Top 10 gaps

| # | Gap | Requirements | Effort on Rome | How we would close it |
|---|---|---|---|---|
| 1 | Declarative parts runtime and component catalog | P-1, P-2, P-5, A-1 | L | Parts become our own services interpreting typed schemas. The Analyst emits configuration, not code. Rome's coding agent stays out of business-user flows. |
| 2 | Several users per workspace, roles and approval limits | R-7.5, R-5.2, P-11, G-6 | L (fork of auth, approvals, actor, people) | An OIDC identity provider (Keycloak, Entra, Google) and a policy service. The approval part runs on Temporal. |
| 3 | Enforced governance: rules, rollout modes, limits, hard floors | G-1, G-3–G-5, R-5.1, R-5.3 | L | A deterministic rules engine (for example GoRules Zen or OPA). Send and write suppression happen at the adapter gateway, not in generated code. |
| 4 | Brief and blueprint as structured, versioned, shared state | R-1.3, R-2.1–R-2.5, A-7 | M | A blueprint store with JSON Schema, versions and diffs, synced to the UI through AG-UI state. |
| 5 | Evaluations on real past cases | R-3.3, A-6, R-6.5 | L | Langfuse datasets and an evaluation runner. The checker is a separate agent. Runs repeat on every version and every document sync. |
| 6 | Security baseline | R-8.4, G-9, G-10, A-5 | L | No shell for business agents, isolated worker pods, a vault, encryption at rest, Presidio, LLM Guard, and Postgres RLS. |
| 7 | Governed integration layer | M-6, G-8, R-7.3 | L | An MCP gateway enforcing read / asks first / never, quotas, test vs real accounts and hidden fields. Adapters for Logo and Netsis, the WhatsApp Cloud API, Teams, and IMAP or Graph mail. |
| 8 | Knowledge parts | P-6, P-7, A-4 | L each | Qdrant with one collection per visibility, a document parser plus a vision model on vLLM, and a metrics layer (for example Cube). |
| 9 | Durable workflows | M-5, P-10, P-11 | M | Temporal workflows, schedules, timers and signals. |
| 10 | Control plane, Turkish, model gateway | M-2, M-4, M-10, R-8.2 | M | Kubernetes provisioning, a `tr` locale, a sweep for locale-aware text handling, LiteLLM, and reply-language control. |

## 6. Risks

| Risk | Finding | Evidence |
|---|---|---|
| License | Rome is MIT. The default runtime depends on a proprietary Anthropic SDK and CLI, and the multi-party control plane is closed. | `packages/core/package.json`, `Dockerfile:240`, `docs/architecture/observability.md` |
| Terms of service | WhatsApp runs through Baileys, an unofficial WhatsApp Web client. Its pinned version is flagged in the lockfile for a message-spoofing advisory (GHSA-qvv5-jq5g-4cgg). WeChat key recovery runs under gdb, and LinkedIn is automated through the browser. None of this is fit for a B2B product in Türkiye. | `packages/core/src/channels/whatsapp.ts:1-13`, `pnpm-lock.yaml:7582-7588`, `docs/architecture/channels.md` |
| Lock-in | The agent loop belongs to Anthropic's or OpenAI's harness. Integrations go through Composio, and identity, OAuth, mail and relay through Rome Cloud. | See M-2, M-4 |
| Maintenance | Public since 22 August 2026, as one squashed commit of 598,811 lines from a private repo. Development runs at about 9 PRs a day and one author wrote 54% of the commits. There have already been breaking format changes (artifact `formatVersion` 2), and the docs have drifted (the observability schema, the lockfile name). | `git log`, `docs/observability/schema.md:107-127` |
| Security | The trust boundary is the whole container: in-container callers count as the guardian, and agents have a shell with `bypassPermissions`. Credentials are stored in plain JSON, the SQLite encryption key is parsed but never used, and one browser is shared. Inbound sender trust relies on LLM triage and attributes that can be changed. Anyone reusing inbound handling should have it reviewed first. | `lib/guardian-session.ts:28-49`, `core/anthropic-provider.ts:548`, `connections/ledger-db.ts:5-8`, `db/index.ts:35-43` |
| Performance and cost | One Node process per workspace, with Chrome, VNC and sshd in an image of about 1.75 GB. Each workspace runs at most 8 action workers by default. SQLite is the default, and Postgres support is nominal: the schema and migrations are SQLite-only (`db/index.ts:52-54`). | `Dockerfile:155-243`, `config.ts:66` |
| i18n | Only English and Simplified Chinese. Turkish case folding and slugs are wrong, currency and locale are hard-coded, and prompts are in English. | See M-10 |

**Modifiability (spec section 9.3)**

| Aspect | Finding |
|---|---|
| Architecture | Modular at the edges: an app declares actions, agents, skills, hooks, web, API and database in a manifest. The core is a monolith: `packages/core/src/index.ts` wires 26 subsystems by hand, and channels and connections can only be added in core. |
| Code quality and tests | Strict TypeScript and 810 test files (about 170k lines of tests against 250k of source). Very large files: `api/routes/webchat.ts` is 3.8k lines and `core/agent-session.ts` 3.6k. |
| Docs | Strong: a concepts index, architecture invariants, 24 ADRs and authoring guides. |
| Community and vendor | 19 authors, and the sponsor is not documented. The monetized pieces (Rome Cloud, the store) are closed. |
| Upstream strategy | Only additive parts could live as apps. Our must-change list touches 8–10 of the 26 core subsystems: auth, approvals, people, connections, channels, mail and OAuth, sessions, prompts, database and dashboard. At the current pace that fork could not be merged back. |
| Data model fit | Solution, blueprint, part, version and address have no counterpart. The guardian is built into people, approvals, memory and prompts. Workspace = instance works. |
| Deployment fit | Docker only, no Helm chart, no GPU components. Compose needs `SYS_ADMIN` and `apparmor:unconfined`. |

## 7. Integration strategy: which pieces to take

Rome is not the base. We build Studio on the stack in section 8 of the spec (LangGraph, Temporal, LiteLLM, MCP, Postgres and Qdrant, Presidio, LLM Guard, Langfuse, Next.js with AG-UI). From Rome we take designs, and code where it stands alone, keeping the MIT notice in `THIRD_PARTY_NOTICES`:

| Piece | Rome source | Used for | Take as |
|---|---|---|---|
| Interaction and suspension protocol: tool-use-id correlation, open state derived from the transcript, dismissal as an outcome | `docs/architecture/suspensions.md`, `app-runtime-sdk/src/index.ts:253-267` | Interrupts, approval cards, "when unsure" asks (G-2, R-4.3, A-2) | Design |
| Question card schema | `core/mcp-facade.ts:1071-1119` | R-1.2, extended with default and "Decide for me" | Design and schema |
| Action result envelope (ok / error / pending approval / pending interaction / handoff) and action manifest (input schema, visibility, `sideEffects`, `requiresApproval`) | `docs/concepts/actions.md`, `app-runtime-sdk/src/index.ts:129-134` | Part tools (A-3), with `sideEffects` actually enforced | Design and types |
| App packaging: manifest, content-addressed atomic installs, one table prefix per package | `docs/concepts/apps.md`, `apps/installer.ts` | Accelerator templates and part packages (R-8.5) | Design |
| Channel ports (send / inbound / accounts / messages), per-conversation ordering, outbox idempotency | `channels/channel.ts`, `docs/concepts/people.md#outbox` | P-3 channels on the WhatsApp Cloud API, Teams and the widget | Design |
| Server-owned connection setup protocol with a generic renderer, and named revocable grants | `docs/architecture/channels.md`, `connections/types.ts` | R-7.3 connect flow, with permission levels and quotas added | Design |
| Session model: fail-closed model pin, isolated vs exact forks, child sessions owning their cost | `docs/concepts/sessions.md`, `core/provider-accounting.ts` | The Analyst's threads, sandbox branches, cost per solution | Design |
| Routine semantics: trigger kinds, fixed vs floating time zones, run history | `docs/concepts/data.md#routines` | P-10 on Temporal Schedules | Design |
| Spec discipline: "ask only for facts only they can supply; never invent", "every workflow must deliver", pin vague quantifiers | `design-workflow/SKILL.md` | The Analyst's R-1 and R-2 behaviour | Prompt patterns |
| Writing discipline: voice guide, concepts index, ADR format | `docs/ui/VOICE.md`, `docs/authoring/` | Our docs and UI copy | Practice |

We do not take:
- the in-container trust model and shell-equipped business agents
- Composio and the Rome Cloud paths
- WeChat, LinkedIn and Baileys automation
- the free-form app builder for business users
- the autonomous memory and skill writers
- the SQLite-first data layer

**Keeping current:** do not fork. Record the evaluated commit. Review Rome's `docs/concepts` and `docs/adrs` each quarter for ideas worth porting.

**If a live demo is needed** before our runtime exists, a one-to-two-week spike can run S-1 on a single Rome instance, with the Analyst as an app agent and approvals by the one guardian. Treat it as throwaway work.

## 8. Vocabulary map

| Rome term | Our term | Note |
|---|---|---|
| Guardian | Builder (owner) | Exactly one per instance |
| Visitor, dashboard visitor | Viewer, approver, IT (partly) | Email allow-lists; signs in through Rome Cloud; no roles |
| Instance + profile | Workspace | One container each |
| Rome Cloud | Infinics control plane and partner console | Closed upstream |
| Main agent (+ planning, assistant, explore) | The Analyst (+ invisible sub-agents) | |
| Rome App | Solution's build output, or a part package | A code package, not a composition of parts |
| Workflow (workflow app) | Assistant flow (P-8) | TypeScript, not a declarative flow |
| Short spec (design-workflow) | Brief + blueprint | Prose, not versioned |
| Action | Part tool, adapter operation | |
| Skill | Analyst know-how, template guidance | |
| Hook | Event subscription | |
| Approval (`requiresApproval`) | "Cannot be skipped" approval | Guardian-only |
| `ask_question`, inline interaction | One question at a time, "when unsure" card | |
| Handoff | Human-in-the-loop hand-over | |
| Routine, `defer` | Schedule (P-10), wake-up | |
| Channel | Chat channel (P-3) | |
| Connection, grant, conferral setup | Connection, permission, connect flow | |
| Connector toolkit (Composio) | Adapter | |
| Memory | Analyst memory | Markdown files |
| Person, account, link, bond level | Contact or customer identity | |
| Policy engine, sentinel, envoy | Inbound routing, injection triage, outbound check | LLM-based |
| Session, agent run, trace, fork | Solution thread, run, agent trace, sandbox branch | |
| Dashboard | Under the hood | |
| App web UI (`/full/apps/<id>`) | Screen or form address | |
| App tables (`<prefix>__*`) | Records storage | No history or roles |
| Lockfile entry | Version (partly) | |
| App Store listing, remix | Infinics catalog, accelerator template | Store backend closed |
| `ctx.dryRun` | Sandbox, shadow | Convention only |
