# Feature: Catalog (solution templates store)

Status: proposal · Label (TR): **Katalog** · Related spec items: R-8.5 (accelerator templates), R-2 (Blueprint), A-7 (everything is a version), G-3 (hard floors), G-8 (read-only by default)

## 1. What it is

The Catalog is a store of ready-made **solution templates** that a workspace can start from. Examples are Invoice Intake, Quote Builder and Return Request Handler.

A business user opens the Catalog, picks a template, and the Analyst turns it into *their* solution. It asks only the questions the template leaves open, then continues through the normal journey: brief, blueprint, build, test, shadow, live.

A workspace can also share one of its own solutions back as a template. It can share only inside the company, or submit it to Infinics for the public catalog.

The idea comes from Rome's App Store. Section 6 says what we keep from Rome and what we change.

## 2. Why it matters

- **Faster start.** "Start from Invoice Intake" beats describing everything from scratch.
- **Infinics' business.** Accelerators become a product line, and partners can package their know-how.
- **Compounding.** A solution proven in one company becomes a starting point for others. The reusable method is shared; the company's data and private rules are not.

## 3. Core concepts

| Term | Meaning | TR label |
|---|---|---|
| Template | A packaged, reusable solution design: parts, flow, rules, approval steps, screens, forms, record schemas, and required connections declared as intent. It is **declarative configuration only**: no code, no customer data, no credentials, no memory. | Şablon |
| Listing | The Catalog identity of a template: name, publisher, description, screenshots, categories. It stays the same across versions. States: `published`, `taken_down`. | Katalog kaydı |
| Version | One immutable package of a template, with a SemVer number, a sha256 content hash, a publish date and a changelog. States: `live`, `superseded`, `revoked`. | Sürüm |
| Publisher | Who owns a listing: Infinics, a partner, or a workspace (for its private catalog). It has a handle, for example `@infinics` or `@karya-home`. | Yayıncı |
| Catalog scope | Where a listing is visible. **Infinics catalog** is public to all workspaces. **Workspace catalog** is private to one company. **Group catalog** is shared across a set of workspaces, for example the companies of one holding. | Katalog kapsamı |
| Install | The link from one workspace solution back to the exact listing, version and hash it started from. | Kurulum |
| Customize | Turning an installed template into the workspace's own solution. The solution keeps its lineage and records where it came from. | Özelleştir |
| Open question | A fact the template needs from each workspace, such as a refund limit, an approver or a mailbox. The Analyst asks it during install, one at a time. | Açık soru |

## 4. User journeys

### 4.1 Install a template (business user)

1. The user opens **Catalog** from Home, browses by category or searches ("iade", "fatura"), and opens a template card. The card shows:
   - what it does, in plain words, with screenshots
   - which parts it uses: "a desk screen for your team, an assistant, an approval step"
   - which connections it needs, and why each one is needed
   - what it will ask you (its open questions)
   - estimated monthly cost and time to set up
   - evidence: test pass rate, and "used by N companies" for public listings
2. **Use this template** starts a Describe conversation with the brief prefilled from the template. The Analyst:
   - asks only the open questions, one at a time, each with a "Decide for me" option
   - fills answers it already knows from Analyst memory and marks them "from memory"
3. The Analyst produces a **blueprint** and shows it as "Template v1.2 plus your changes".
4. From here the normal journey applies: approve, connect systems, build, test on past cases, try, launch.

A template never goes live directly. Customer-facing templates default to shadow mode.

### 4.2 Update an installed template

- When a new version of the listing is published, the solution shows **Update available** with a readable blueprint diff and the changelog. The workspace never updates automatically.
- Applying an update creates a **draft version** of the solution. Local customizations are kept through a three-way merge at blueprint level: template base, new template version, the workspace's solution.
- The draft is tested before it can go live, like any other change (R-6.3).
- If the update asks for more access, such as a new write adapter or a new system, it is flagged before approval (R-2.5).

### 4.3 Share a solution as a template (builder, IT or partner)

1. From a live solution, choose **Share as template**. Pick the scope: this workspace, the group, or submit to the Infinics catalog.
2. The Analyst separates the reusable method from private context:
   - Company facts become **open questions**: limits, names, mailboxes, approvers.
   - Customer data, memory, credentials and test cases are removed. Only the test-set *schema* travels, never the cases.
   - A PII scan runs on every text field of the package. A hit blocks publishing until it is resolved.
3. The builder reviews the store card. Copy is in Turkish and English, and the Analyst drafts both.
4. **Approval** depends on the scope:
   - Workspace scope needs the workspace owner.
   - Group scope needs the owner of each receiving workspace to enable the group catalog once.
   - The Infinics catalog needs an Infinics reviewer. The reviewer checks the blueprint, the requested access, the PII scan and the test evidence.
5. Publishing creates a new immutable version. The first publish also creates the listing.

### 4.4 Take down or revoke (publisher or Infinics)

- **Take down a listing:** it disappears from browsing. Existing installs keep working and keep their history.
- **Revoke a version:** no new installs of that version. Every solution installed from it shows a warning, and its owner gets an attention item recommending an update or a pause.
- Pausing a solution is a decision that outlives the conversation. The Catalog therefore never pauses a customer's solution on its own (G-3). The one exception is an Infinics security revocation, which may pause a solution. It is logged, the owner is told immediately, and the owner can resume it.

## 5. Rules (contracts)

1. **Versions are immutable.** A published version's content and hash never change. A fix is always a new version.
2. **SemVer is monotonic per listing.** A new version must be greater than every earlier one.
3. **Nothing is deleted.** Superseded and revoked versions are kept forever, so every install can be traced and reproduced.
4. **Installs pin an exact version and hash.** No install ever points at "latest". The install verifies the sha256 and the publisher signature before anything is read.
5. **Updates are opt-in.** The workspace never updates in the background.
6. **A template is data, not code.** It contains only schemas from the controlled parts vocabulary (spec A-1): screen and form definitions from the component catalog, record schemas, rules for the rules engine, flow definitions, and connection intents. It never contains executable code. This keeps packages reviewable and safe to install.
7. **Connections are declared intent, not access.** A template says "needs: Logo (read orders), Mail (read inbox)". Installing grants nothing. Each connection is granted by IT through the normal Connections flow, read-only by default (G-8).
8. **Templates carry no workspace data.** No customer data, memory, credentials or past cases. A PII scan enforces this at publish time.
9. **Lineage is recorded once, on the solution.** The solution stores `{listingId, version, contentHash}` for its origin and the last update it applied. There is no second lineage record elsewhere.
10. **Scopes are enforced by the Catalog service, not by the UI or the model.** A workspace can list and install only what its scopes allow.
11. **The listing ID is the template ID.** `@infinics/invoice-intake` stays that exact string everywhere. The publisher handle `infinics` is reserved for first-party templates.

## 6. What we keep from Rome, and what we change

| Rome's App Store (evaluated at `rome-os/rome@ef523c4`) | Our Catalog |
|---|---|
| Listing and version are separate objects (`docs/concepts/apps.md#app-store`) | **Keep** |
| Immutable versions, monotonic SemVer, full retention | **Keep** |
| Install pins `listingId + version + contentHash` (`packages/api-types/src/apps.ts`, `AppstoreSourceSchema`) | **Keep**, and add a publisher signature |
| Updates are opt-in; the consumer never polls | **Keep** |
| Scoped IDs `@handle/slug`, with `rome` reserved | **Keep**, with `infinics` reserved |
| Remix: new ID, lineage recorded on the copy (`app.yaml#remix`) | **Keep**, as Customize with lineage on the solution |
| Store card file with title, description, categories, keywords and media (`.rome_store/rome_store.yaml`) | **Keep**, plus Turkish and English copy, connections needed, open questions, cost and evidence |
| Packages are **code** (TypeScript actions, React UI) | **Change**: packages are declarative templates only (rule 6) |
| Installing makes the app live immediately | **Change**: installing starts the journey, and the solution goes live only after build, tests and launch |
| One global store hosted in Rome Cloud (closed source) | **Change**: our own Catalog service in Türkiye with workspace, group and Infinics scopes. On-prem customers get a signed offline mirror |
| No review step before listing | **Change**: an Infinics reviewer for the public catalog, and an owner approval for private scopes |
| No data hygiene on publish | **Change**: stripping private context plus a PII scan |

## 7. Package format (sketch)

```yaml
# template.yaml — declarative only
id: "@infinics/return-request-handler"
version: 1.2.0
name: { en: Return Request Handler, tr: İade Talebi Yöneticisi }
summary: { en: "Answers return requests by email and WhatsApp…", tr: "…" }
parts:
  - { kind: assistant, id: returns_assistant }
  - { kind: screen, id: returns_desk, schema: screens/returns-desk.json }
  - { kind: records, id: return_cases, schema: records/return-cases.json }
  - { kind: approval, id: refund_approval, enforced: true }
rules:
  - { id: fourteen_day_policy, mode: exact, source: rules/fourteen-day.json }
  - { id: campaign_items_exchange_only, mode: exact, param: campaign_tag }
  - { id: tone_and_reply, mode: judgment }
open_questions:
  - { id: refund_limit, ask: { en: "Above what amount should a refund need approval?", tr: "…" }, type: money, default: "1000 TRY" }
  - { id: approver, ask: { en: "Who approves refunds above the limit?", tr: "…" }, type: person }
connections:
  - { system: logo, access: read, why: { en: "to find the order", tr: "…" } }
  - { system: mail, access: read, why: { en: "to read return requests", tr: "…" } }
  - { system: whatsapp_business, access: send, why: { en: "to reply to customers", tr: "…" } }
test_set_schema: tests/return-case.schema.json   # schema only, never cases
launch_default: shadow
catalog: catalog/card.yaml                        # store card: copy, categories, keywords, media
```

The package is a tarball of this directory. Its sha256 is the content hash, and the publisher's signature covers that hash.

## 8. Data model (sketch)

| Table | Key columns |
|---|---|
| `publishers` | `id`, `handle` (unique), `kind` (infinics / partner / workspace), `owner_workspace_id?`, `signing_key` |
| `listings` | `id` (= `@handle/slug`), `publisher_id`, `scope` (infinics / group / workspace), `scope_ref`, `state`, `card` (JSON) |
| `listing_versions` | `listing_id`, `version`, `content_hash`, `signature`, `state`, `changelog`, `published_at`, `published_by`, `review_id?`; unique (`listing_id`, `version`) |
| `reviews` | `id`, `version_ref`, `reviewer`, `decision`, `notes`, `checks` (PII scan, access delta, test evidence) |
| `installs` | `workspace_id`, `solution_id`, `listing_id`, `version`, `content_hash`, `installed_at`, `installed_by` |
| `catalog_access` | `workspace_id`, `scope`, `scope_ref`, `enabled_by`, `enabled_at` |

Solutions keep `origin: {listingId, version, contentHash}` and `last_applied_update` (rule 9).

## 9. Service API (sketch)

| Call | Who | Does |
|---|---|---|
| `GET /catalog/listings?q=&category=&scope=` | workspace users | Lists what the workspace's scopes allow |
| `GET /catalog/listings/{id}` and `…/versions` | workspace users | Card, versions, changelog, evidence |
| `POST /catalog/installs` `{listingId, version}` | builder | Verifies hash and signature, then opens a prefilled Describe thread and returns the solution draft |
| `GET /solutions/{id}/updates` | builder | Newer versions, with a blueprint diff and any access increase |
| `POST /solutions/{id}/updates` `{version}` | builder | Creates a draft solution version through the three-way merge |
| `POST /catalog/submissions` `{solutionId, scope}` | builder, IT, partner | Strips private context, runs the PII scan, and creates a pending version |
| `POST /catalog/reviews/{id}` `{decision}` | owner or Infinics reviewer | Publishes or rejects |
| `POST /catalog/listings/{id}/takedown`, `POST /catalog/versions/{v}/revoke` | publisher, Infinics | Takedown or revocation, following section 4.4 |

The Analyst reaches the Catalog through tools (A-3): `catalog.search`, `catalog.read`, `catalog.install`. It can *suggest* a template ("There is an Invoice Intake template; start from it?"). It can never publish, approve, take down or revoke anything (G-3).

## 10. Screens

- **Catalog** (Home → Catalog): search, categories, scope filter, template cards.
- **Template page:** the card, "What it will ask you", connections needed, versions and changelog, **Use this template**.
- **Solution → Origin:** where this solution came from, update available, blueprint diff, **Review update**.
- **Share as template:** scope choice, stripped fields, PII scan result, card editor, submit.
- **Review queue:** for owners and Infinics reviewers, showing the checks from section 4.3.
- **Under the hood:** package hash, signature, raw `template.yaml`, install log.

## 11. Phasing

| Phase | Scope |
|---|---|
| Phase 2 | Infinics catalog, read-only. Install accelerator templates with open questions and lineage. Update available, with a diff. Two first templates: Return Request Handler (S-1) and Price List from Excel (S-3). |
| Phase 3 | Workspace and group catalogs. Share as template with stripping and PII scan. Review queue. Revocation. Signed offline mirror for on-prem. |
| Later | Partner publishers, adapter listings (MCP adapters that IT approves), and paid templates with billing. |

## 12. Acceptance criteria

1. Installing `@infinics/return-request-handler@1.2.0` asks only its open questions, marks answers taken from memory, and produces a blueprint labelled "Template v1.2.0 plus your changes". Nothing goes live until the solution passes its tests and is launched in shadow mode.
2. A tampered package, one whose hash or signature does not match, is refused before any content is read. The refusal is logged.
3. Publishing an existing version number or a lower one is refused. A published version's bytes can never change.
4. An update that adds a write connection is flagged as an access increase before approval. Applying it creates a draft version that is tested before going live.
5. Share as template on a solution containing a customer name, phone number or tax number is blocked by the PII scan until those values are removed.
6. A workspace cannot see or install a listing outside its scopes, even by calling the API directly with the listing ID.

## 13. Open questions

- Who may publish to the group catalog: the group's holding workspace only, or any member?
- Do partners get revenue share on paid templates, and does billing count per install or per active solution?
- Should the evidence on a public card ("used by N companies", pass rate) be opt-in for the companies that contributed it?
- How long does a revoked version keep running before the owner must act?
- Are adapters (MCP connectors) listed in the same Catalog, or in a separate IT-only directory?
